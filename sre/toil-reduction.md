# Toil Reduction

## Definition (per Google SRE)
Toil = manual, repetitive, automatable, no lasting value, scales linearly

## Measurement
```
Toil % = toil_hours / total_hours × 100
Target: < 50% (Google SRE book)
Our target: < 30%
```

## Top toil sources (Q4 tracking)
1. Certificate rotation — automate with cert-manager
2. DB user provisioning — automate with Vault
3. Environment setup — automate with Backstage templates
4. Incident triage — automate with PagerDuty event rules
