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

## Updated: real-world examples
- Note added 2025-12-14

## Updated: security hardening
- Note added 2025-12-15

## Updated: security hardening
- Note added 2025-12-15

## Updated: real-world examples
- Note added 2025-12-22

## Updated: monitoring tips
- Note added 2025-12-26

## Updated: real-world examples
- Note added 2026-01-06

## Updated: troubleshooting steps
- Note added 2026-01-11

## Updated: monitoring tips
- Note added 2026-02-04

## Updated: production patterns
- Note added 2026-02-07

## Updated: real-world examples
- Note added 2026-02-27

## cleanup and reformat
- Updated 2026-03-07
