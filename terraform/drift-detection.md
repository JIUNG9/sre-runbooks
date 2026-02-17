# Drift Detection

## Scheduled plan
```bash
#!/bin/bash
terraform plan -detailed-exitcode -out=plan.out
EXIT_CODE=$?
if [ $EXIT_CODE -eq 2 ]; then
  echo "DRIFT DETECTED" | slack-notify
fi
```

## Common drift sources
1. Manual console changes (tag edits, security group rules)
2. AWS auto-modifications (RDS maintenance, EKS add-on updates)
3. ASG scaling events
4. CloudFormation-managed resources overlapping

## Prevention
- SCP to deny console access to IaC-managed resources
- Nightly plan cron job

## Updated: performance tuning
- Note added 2026-01-13

## Updated: production patterns
- Note added 2026-02-17
