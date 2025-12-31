# K8s DNS Troubleshooting

## Quick checks
```bash
kubectl run dnstest --image=busybox:1.36 --rm -it -- nslookup kubernetes.default
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

## Common issues
1. CoreDNS pods crashlooping - check ConfigMap
2. ndots:5 causing slow resolution - set ndots:2 in pod spec
3. Search domain expansion - FQDN with trailing dot
4. NodeLocalDNS cache for high-QPS services

## Updated: production patterns
- Note added 2025-12-31
