# Pod Disruption Budgets

Ensure availability during voluntary disruptions.

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: api-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: api
```

Key points:

- PDBs only affect voluntary disruptions (drain, rolling update)
- involuntary disruptions (node crash) are not affected
- maxUnavailable vs minAvailable - pick one
