# Terraform Import Workflow

## 1.5+ import blocks

```hcl
import {
  to = aws_instance.web
  id = "i-0123456789abcdef0"
}
```

Then: `terraform plan -generate-config-out=generated.tf`

## Manual import

```bash
terraform import aws_instance.web i-0123456789abcdef0
terraform plan  # verify no drift
```

## Tips

- Always `plan` after import to verify state matches
- Some resources require composite IDs (e.g., `subnet_id/association_id`)
