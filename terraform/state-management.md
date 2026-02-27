# Terraform State Management

## S3 Backend with Native Locking (1.10+)

```hcl
terraform {
  backend "s3" {
    bucket       = "tfstate-prod"
    key          = "infra/terraform.tfstate"
    region       = "ap-northeast-2"
    use_lockfile = true
  }
}
```

No more DynamoDB lock table needed since Terraform 1.10.

## State isolation
- Per-environment state files
- Per-service state files
- Use `terraform_remote_state` data source sparingly

## Updated: monitoring tips
- Note added 2026-02-27
