# Node Affinity & Taints

## Spread workloads across AZs

```yaml
topologySpreadConstraints:
- maxSkew: 1
  topologyKey: topology.kubernetes.io/zone
  whenUnsatisfiable: DoNotSchedule
  labelSelector:
    matchLabels:
      app: api
```

## Taint/Toleration for dedicated nodes
```bash
kubectl taint nodes gpu-node-1 gpu=true:NoSchedule
```

## Updated: production patterns
- Note added 2026-01-02

## Updated: troubleshooting steps
- Note added 2026-01-15
