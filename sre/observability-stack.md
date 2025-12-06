# Observability Stack

## Three pillars
```
Metrics → Prometheus/SigNoz → dashboards, alerts
Logs → OpenTelemetry → SigNoz/ELK → search, analysis
Traces → OpenTelemetry → SigNoz → latency debugging
```

## SigNoz setup notes
- ClickHouse backend for metrics + traces
- OpenTelemetry Collector as DaemonSet
- Retention: 15d metrics, 7d traces, 30d logs
- Alert rules via YAML, managed in Git

## Key dashboards
1. Service overview (RED metrics)
2. Infrastructure (node CPU/mem/disk)
3. SLO burn rate
