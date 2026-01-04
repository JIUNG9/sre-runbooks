# IAM Best Practices

## Least privilege pattern
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:GetObject",
      "s3:ListBucket"
    ],
    "Resource": [
      "arn:aws:s3:::my-bucket",
      "arn:aws:s3:::my-bucket/*"
    ],
    "Condition": {
      "StringEquals": {
        "aws:PrincipalOrgID": "o-xxxxx"
      }
    }
  }]
}
```

## Key rules
1. No inline policies — use managed policies
2. Require MFA for all human users
3. Use OIDC federation for CI/CD (no long-lived keys)
4. SCPs at org level for guardrails
