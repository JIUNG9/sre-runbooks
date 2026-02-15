# HPA Tuning Notes

## CPU-based scaling

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
```

## Custom metrics
- KEDA for event-driven scaling
- Prometheus adapter for custom metric HPA

## Updated: real-world examples
- Note added 2026-02-09

## Updated: security hardening
- Note added 2026-02-14

## Updated: troubleshooting steps
- Note added 2026-02-15
