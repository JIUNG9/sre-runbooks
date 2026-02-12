# CI/CD Pipeline Security

## Supply chain hardening
1. Pin action versions to SHA (not tags)
   ```yaml
   - uses: actions/checkout@8ade135a41bc03ea155e62e844d188df1ea18608  # v4.1.0
   ```
2. OIDC for cloud auth (no stored credentials)
3. SLSA provenance for container images
4. Dependabot + Renovate for dependency updates

## Secrets hygiene
- Never echo secrets in logs
- Use GitHub environment protection rules
- Rotate secrets quarterly
- Audit with `gh secret list`

## Updated: monitoring tips
- Note added 2026-02-13
