# Terraform Testing

## Native test framework (1.6+)

```hcl
# tests/vpc.tftest.hcl
run "create_vpc" {
  command = apply

  assert {
    condition     = aws_vpc.main.cidr_block == "10.0.0.0/16"
    error_message = "VPC CIDR mismatch"
  }
}
```

## Validation
```hcl
variable "instance_type" {
  validation {
    condition     = can(regex("^t3\\.", var.instance_type))
    error_message = "Only t3 instances allowed."
  }
}
```

## Updated: troubleshooting steps
- Note added 2026-02-15
