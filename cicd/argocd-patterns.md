# ArgoCD Patterns

## ApplicationSet for multi-env

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: api-service
spec:
  generators:
  - list:
      elements:
      - env: dev
        cluster: dev-cluster
      - env: prod
        cluster: prod-cluster
  template:
    spec:
      source:
        repoURL: https://github.com/org/k8s-manifests
        path: 'apps/api/{{env}}'
      destination:
        server: '{{cluster}}'
```

## Sync waves

- Wave 0: Namespaces, CRDs
- Wave 1: ConfigMaps, Secrets
- Wave 2: Deployments
- Wave 3: Services, Ingress
