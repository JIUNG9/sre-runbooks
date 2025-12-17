# Resource Quotas & LimitRanges

## Namespace-level quotas

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"
```

## LimitRange for defaults

Always set default limits to prevent OOMKill cascades.
