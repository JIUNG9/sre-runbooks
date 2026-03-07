# RBAC Patterns

## Least-privilege service accounts

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

## Cross-namespace access
- ClusterRole + RoleBinding = namespace-scoped cluster role
- ClusterRole + ClusterRoleBinding = cluster-wide
- Avoid wildcard verbs in production

## Updated: production patterns
- Note added 2026-02-01

## Updated: real-world examples
- Note added 2026-02-25

## Updated: monitoring tips
- Note added 2026-02-26

## note from postmortem review
- Updated 2026-03-07
