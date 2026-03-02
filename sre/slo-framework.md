# SLO Framework

## The hierarchy
```
SLI (indicator) → SLO (objective) → SLA (agreement)
```

## Key SLIs
| Service Type | SLI | Measurement |
|-------------|-----|-------------|
| API | Availability | successful requests / total requests |
| API | Latency | p99 response time |
| Pipeline | Freshness | time since last successful run |
| Storage | Durability | successful reads / total reads |

## Error budget
```
Error budget = 1 - SLO target
99.9% SLO → 0.1% error budget → 43.8 min/month
```

## Updated: production patterns
- Note added 2025-12-20

## Updated: best practices
- Note added 2026-02-07

## Updated: performance tuning
- Note added 2026-02-08

## add troubleshooting tip
- Updated 2026-03-02
