# etcd Maintenance

## Defragmentation
```bash
etcdctl defrag --endpoints=https://etcd-0:2379
```

## Backup
```bash
etcdctl snapshot save /backup/etcd-$(date +%Y%m%d).db
etcdctl snapshot status /backup/etcd-*.db --write-out=table
```

## Monitoring
- etcd_server_has_leader
- etcd_disk_backend_commit_duration_seconds
- etcd_network_peer_round_trip_time_seconds
