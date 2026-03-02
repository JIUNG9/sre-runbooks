# GitHub Actions Patterns

## Reusable workflow
```yaml
# .github/workflows/reusable-deploy.yml
on:
  workflow_call:
    inputs:
      environment:
        type: string
        required: true
    secrets:
      AWS_ROLE_ARN:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
    steps:
    - uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
    - run: terraform apply -auto-approve
```

## Self-hosted runners (ARC)
- Actions Runner Controller on EKS
- Ephemeral runners for security
- Scale-to-zero between jobs

## Updated: best practices
- Note added 2025-12-14

## Updated: best practices
- Note added 2026-01-17

## Updated: performance tuning
- Note added 2026-01-29

## update for latest version
- Updated 2026-03-02
