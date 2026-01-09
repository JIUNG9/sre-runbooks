# Secrets Management

## External Secrets Operator (ESO)

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: db-credentials
  data:
  - secretKey: password
    remoteRef:
      key: prod/db/password
```

## Key rotation
- Lambda function triggered by Secrets Manager rotation
- RDS auto-rotation for managed credentials
- ESO auto-refreshes K8s secrets
