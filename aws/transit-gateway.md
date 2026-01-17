# Transit Gateway Architecture

## Hub-spoke pattern
```
         ┌─────────┐
    ┌────┤   TGW   ├────┐
    │    └────┬────┘    │
    │         │         │
┌───┴──┐ ┌───┴──┐ ┌───┴──┐
│Shared │ │ Dev  │ │ Prod │
│  VPC  │ │ VPC  │ │ VPC  │
└──────┘ └──────┘ └──────┘
```

## Route table isolation
- Shared services: route to all VPCs
- Dev: route to shared only
- Prod: route to shared only
- Dev ↔ Prod: NO direct route

## Cost
- $0.05/hr per attachment
- $0.02/GB data processing
- Cheaper than VPC peering at scale (>3 VPCs)
