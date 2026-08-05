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
