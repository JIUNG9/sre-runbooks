# Runbook Template

## {{SERVICE_NAME}} - {{ALERT_NAME}}

### Overview
- What this alert means
- Expected impact
- SLO affected

### Diagnosis
```bash
# Step 1: Check service health
kubectl get pods -n {{namespace}} -l app={{service}}

# Step 2: Recent logs
kubectl logs -n {{namespace}} -l app={{service}} --since=15m

# Step 3: Check dependencies
curl -s http://{{service}}/health | jq
```

### Mitigation
1. Scale up: `kubectl scale deployment/{{service}} --replicas=5`
2. Restart: `kubectl rollout restart deployment/{{service}}`
3. Rollback: `kubectl rollout undo deployment/{{service}}`

## Updated: production patterns
- Note added 2026-02-01

## Updated: production patterns
- Note added 2026-02-13

## Updated: troubleshooting steps
- Note added 2026-02-20

## Updated: performance tuning
- Note added 2026-03-04

## security hardening note
- Updated 2026-03-17
