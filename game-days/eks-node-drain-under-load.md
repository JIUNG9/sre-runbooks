# Game Day: Node Drain Under Load on EKS

A worked example of [`game-day-template.md`](game-day-template.md). The numbers are
illustrative; replace them with your own SLO and measurements.

## Scope

- **System under test:** `checkout-api`, a stateless HTTP service, 6 replicas,
  in a staging EKS cluster.
- **Failure:** `kubectl drain` of one node that hosts 2 of the 6 replicas, while a
  load generator sends 200 requests per second.
- **Window:** 60 minutes, hard stop at the end.
- **Out of scope:** the database, the ingress controller, node group scaling.

## Hypothesis

```text
When one node is drained under 200 req/s,
the success rate stays above 99.5% of requests,
p99 latency stays under 800 ms,
and no request fails because a pod was killed mid-request.
```

This tests three assumptions that are often only assumed: the
[PodDisruptionBudget](../kubernetes/pod-disruption-budgets.md) holds, replicas are
spread across nodes, and the app drains connections on SIGTERM.

## Preconditions

Check these before starting. If any fails, fix it first and run the day later.

```bash
# The PDB exists and allows at least one disruption
kubectl get pdb -n {{namespace}} checkout-api
# ALLOWED DISRUPTIONS must be >= 1

# Replicas are spread over more than one node
kubectl get pods -n {{namespace}} -l app=checkout-api -o wide

# The cluster has room to reschedule the evicted pods
kubectl describe node {{node}} | grep -A6 'Allocated resources'

# Grace period is longer than the longest request
kubectl get deploy checkout-api -n {{namespace}} \
  -o jsonpath='{.spec.template.spec.terminationGracePeriodSeconds}'
```

## Blast radius

| Question | Answer |
| --- | --- |
| Who sees impact? | Load generator traffic only; staging has no users |
| Other callers of `checkout-api`? | None in staging |
| Traffic share affected | 2 of 6 replicas (33%) briefly |
| Shared infrastructure | One node in a shared node group; other teams' pods on it are evicted too |

The last row matters: draining a node evicts every pod on it. Pick a node that hosts
only this team's workloads, or tell the other teams before the day.

## Abort criteria

| Condition | Action |
| --- | --- |
| Success rate below 95% for 2 minutes | `kubectl uncordon {{node}}`, stop the load |
| Drain blocked for more than 10 minutes | Stop, record which PDB or pod blocked it |
| Pods stay `Pending` for more than 5 minutes | `kubectl uncordon {{node}}` |

Rollback is `kubectl uncordon {{node}}`. It does not move pods back, so also run
`kubectl rollout restart deployment/checkout-api -n {{namespace}}` if spread is
uneven afterwards.

## Procedure

1. Start the load generator and wait 5 minutes for a steady baseline. Record the
   success rate and p99.
2. Pick the node and confirm it hosts 2 replicas.

   ```bash
   kubectl get pods -n {{namespace}} -l app=checkout-api \
     -o custom-columns=POD:.metadata.name,NODE:.spec.nodeName
   ```

3. Drain it and record the start time.

   ```bash
   kubectl drain {{node}} --ignore-daemonsets --delete-emptydir-data --timeout=600s
   ```

4. Watch the replacement pods and the error rate until the deployment is back to 6
   ready replicas.

   ```bash
   kubectl rollout status deployment/checkout-api -n {{namespace}}
   ```

5. Hold for 10 minutes, then run the rollback: `kubectl uncordon {{node}}`.
6. Stop the load and export the metrics for the window.

## What to look for

| Signal | Healthy | Problem it points to |
| --- | --- | --- |
| Drain finishes within a few minutes | Yes | A PDB with `minAvailable` equal to replicas blocks the drain forever |
| Errors only at the start of the drain | Short burst of 502/503 | Readiness probe slower than the termination; no `preStop` delay |
| Errors for the whole drain | Sustained | Both evicted replicas were behind one load balancer target group that deregisters slowly |
| Replacement pods `Pending` | None | No spare capacity, or a `nodeSelector` / topology constraint with no valid node |
| Replacement pods land on one node | Spread across nodes | Missing `topologySpreadConstraints`; the next drain takes more replicas |
| Connection resets at pod stop | None | The app ignores SIGTERM, or `terminationGracePeriodSeconds` is shorter than request time |

A common fix for the first burst of errors is a short `preStop` sleep, so the load
balancer deregisters the pod before the process receives SIGTERM:

```yaml
lifecycle:
  preStop:
    exec:
      command: ["sleep", "10"]
```

## Example results

Illustrative values for a first run. A first run usually finds a gap.

| Measure | Hypothesis | Observed |
| --- | --- | --- |
| Success rate during drain | above 99.5% | 98.7% |
| p99 latency | under 800 ms | 1,240 ms for 40 seconds |
| Drain duration | not stated | 3 min 10 s |
| Alert fired | within 5 minutes | not fired |

The hypothesis failed. The 1.3% errors came in the first 15 seconds, when the load
balancer still routed to pods that had received SIGTERM. The latency alert threshold
was above the 40-second spike, so no alert fired.

## Example action items

| Item | Owner | Due |
| --- | --- | --- |
| Add a 10 second `preStop` sleep to the deployment | {{owner}} | {{date}} |
| Add `topologySpreadConstraints` over `kubernetes.io/hostname` | {{owner}} | {{date}} |
| Lower the latency alert to `for: 30s` | {{owner}} | {{date}} |
| Re-run this exercise after the changes | {{owner}} | {{date}} |
