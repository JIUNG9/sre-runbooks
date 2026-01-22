# Capacity Planning

## Load testing setup
```bash
# k6 script skeleton
import http from 'k6/http';
import { check } from 'k6';

export let options = {
  stages: [
    { duration: '5m', target: 100 },
    { duration: '10m', target: 500 },
    { duration: '5m', target: 0 },
  ],
};
```

## Headroom formula
```
Required capacity = peak_load × 1.5 (headroom) × 1.2 (growth)
N+1 redundancy for stateful services
N+2 for cross-AZ resilience
```
