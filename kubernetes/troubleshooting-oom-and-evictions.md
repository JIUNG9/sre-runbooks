# Troubleshooting OOMKilled and Evictions

A pod that disappears for memory reasons ends in one of two ways, and the fix differs.
OOMKilled means the container went over its own limit. An eviction means the node ran
low and the kubelet removed pods to recover. Decide which one it was before changing
anything.

## Tell them apart

| | OOMKilled | Node-pressure eviction |
| --- | --- | --- |
| Who acts | Kernel OOM killer, inside the container's cgroup | Kubelet |
| Trigger | Container memory reached `limits.memory` | Node crossed an eviction threshold (`memory.available`, `nodefs.available`, ...) |
| Pod phase | Stays in the pod; container restarts per `restartPolicy` | Pod ends as `Failed`, reason `Evicted`; a controller creates a replacement |
| Exit code | 137 | None (the pod is terminated by the kubelet) |
| Where to look | `kubectl describe pod`, container `Last State` | `kubectl describe pod`, pod `Status` / `Message`; node conditions |

```bash
# OOMKilled: last terminated state of each container
kubectl get pod {{pod}} -n {{namespace}} \
  -o jsonpath='{range .status.containerStatuses[*]}{.name}{"\t"}{.lastState.terminated.reason}{"\t"}{.lastState.terminated.exitCode}{"\n"}{end}'

# Evicted pods in the namespace, with the kubelet's message
kubectl get pods -n {{namespace}} --field-selector=status.phase=Failed \
  -o custom-columns=NAME:.metadata.name,REASON:.status.reason,MESSAGE:.status.message
```

Exit code 137 is `128 + SIGKILL`. It also appears when something else sends
SIGKILL, such as a liveness probe that exceeds `terminationGracePeriodSeconds`, so
confirm `reason: OOMKilled` and don't rely on the code alone.

## OOMKilled

`kubectl describe pod {{pod}}` shows:

```text
Last State:     Terminated
  Reason:       OOMKilled
  Exit Code:    137
```

Confirm on the node. The kernel log names the cgroup that hit its limit:

```bash
dmesg -T | grep -i -E 'out of memory|oom-kill|killed process'
# Memory cgroup out of memory: Killed process 4121 (java) ... oom_memcg=/kubepods/burstable/pod<uid>/<container-id>
```

`oom_memcg` is the container's cgroup. If it is the pod's own cgroup, the container
exceeded its limit. If the message says `Out of memory:` without a memcg and the node
is full, the kernel killed something to save the node, and you should read the
eviction section instead.

Things that commonly cause it:

- The limit is lower than the working set. Compare against the real peak, not the
  average: `max_over_time(container_memory_working_set_bytes{pod="{{pod}}"}[7d])`.
- A runtime sized itself from the node and ignored the limit. A JVM older than 8u191
  reads host memory; newer ones respect the cgroup. Set `-XX:MaxRAMPercentage`
  explicitly, and remember heap is only part of the footprint (metaspace, thread
  stacks, direct buffers).
- Page cache counted against the cgroup. `container_memory_working_set_bytes` already
  excludes inactive cache and is the figure the OOM killer is closest to. Don't alert
  on `container_memory_usage_bytes`, which includes reclaimable cache.
- A leak. Working set climbs steadily between restarts and drops to zero at each one.

Fixes, in order of preference: fix the leak or the runtime sizing, then raise the limit.
Raising the limit only moves the kill later when the cause is a leak.

## Node-pressure eviction

```bash
kubectl describe node {{node}} | grep -A8 Conditions
# MemoryPressure   True   KubeletHasInsufficientMemory
# DiskPressure     True   KubeletHasDiskPressure

kubectl get events -A --field-selector reason=Evicted
```

The evicted pod's `status.message` states the resource and the numbers, for example
`The node was low on resource: memory. Threshold quantity: 100Mi, available: 85Mi.
Container api was using 2Gi, request is 512Mi, has larger consumption of memory.`

The kubelet picks victims by comparing usage to requests, then by priority:

1. Pods using more than their request, ordered by how far over.
2. Lower `priorityClass` first among those.
3. Pods using less than their request go last.

This is why a pod with a small request and a large real footprint is evicted first,
and why a quiet, well-requested pod is usually safe.

Default hard thresholds are `memory.available<100Mi`, `nodefs.available<10%`,
`imagefs.available<15%`. Check the kubelet config for your values:

```bash
kubectl get --raw "/api/v1/nodes/{{node}}/proxy/configz" | jq '.kubeletconfig.evictionHard'
```

Fixes: set requests close to real usage so the scheduler stops overpacking the node,
reserve resources for the system with `kubeReserved` and `systemReserved`, and spread
heavy pods across nodes. Disk pressure from logs or `emptyDir` needs
`ephemeral-storage` requests and limits.

## Limits vs requests

- `requests.memory` is what the scheduler reserves and what eviction ranking uses.
- `limits.memory` is what the kernel enforces. Over it, the container is killed.
- Requests lower than limits make a pod `Burstable`. It can use spare node memory, and
  it is also first in line when the node runs out.
- Requests equal to limits make it `Guaranteed`. It is evicted last and gets a lower
  `oom_score_adj`, so the kernel prefers other victims.
- No requests and no limits is `BestEffort`: evicted first, killed first.

For memory, setting request equal to limit is the safest default. Memory can't be
reclaimed from a running process the way CPU time can be throttled, so burst headroom
turns into a kill when the node is full.

## Triage order

1. `kubectl describe pod`: is it `OOMKilled` (container) or `Evicted` (pod)?
2. OOMKilled: compare the limit with the peak working set, then check runtime sizing.
3. Evicted: read the node conditions and the eviction message, then compare requests
   with real usage on that node.
4. Neither shows in `describe`: check `dmesg` on the node. The pod may have been
   killed by the kernel after its container runtime was already gone.
