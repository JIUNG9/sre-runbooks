# Container Security

## Image scanning
```bash
# Trivy
trivy image --severity HIGH,CRITICAL myapp:latest

# Grype
grype myapp:latest
```

## Best practices
1. Non-root user: `USER 1000:1000`
2. Read-only filesystem: `--read-only`
3. Drop all capabilities: `--cap-drop=ALL`
4. No new privileges: `--security-opt=no-new-privileges`
5. Resource limits: `--memory=512m --cpus=0.5`

## update for latest version
- Updated 2026-04-11
