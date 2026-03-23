# Terraform Module Patterns

## Composition over inheritance

```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr   = var.vpc_cidr
  azs    = var.availability_zones
}

module "eks" {
  source     = "./modules/eks"
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnet_ids
}
```

## Module versioning
- Use git tags for internal modules
- Pin to exact versions in production
- `source = "git::https://github.com/org/module.git?ref=v1.2.3"`

## Updated: monitoring tips
- Note added 2026-01-29

## Updated: best practices
- Note added 2026-01-31

## added monitoring query
- Updated 2026-03-23
