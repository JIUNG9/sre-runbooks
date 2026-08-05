# AWS Cost Optimization Checklist

## Quick wins

- [ ] Delete unattached EBS volumes
- [ ] Remove unused Elastic IPs
- [ ] Clean up old snapshots (>90d)
- [ ] Right-size RDS instances (check CPU <20%)
- [ ] Review NAT Gateway data processing costs
- [ ] S3 lifecycle policies for all buckets
- [ ] GP3 migration from GP2 (20% cheaper)

## Reserved/Savings Plans

- Compute SP for flexible workloads
- EC2 RI for stable baseline
- RDS RI for production databases
- 1-year no-upfront for flexibility
