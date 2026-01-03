# Kubernetes Network Policies

## Ingress Rules

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {{}}
  policyTypes:
  - Ingress
```

## Egress Rules

- Default deny egress blocks all outbound traffic
- Combine with DNS allow for practical use
