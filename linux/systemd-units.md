# systemd Unit Patterns

## Service with restart
```ini
[Unit]
Description=API Server
After=network.target

[Service]
Type=notify
ExecStart=/usr/bin/api-server
Restart=on-failure
RestartSec=5
WatchdogSec=30
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

## Hardening
```ini
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
NoNewPrivileges=true
ReadWritePaths=/var/lib/api
```

## Updated: security hardening
- Note added 2026-02-04

## Updated: real-world examples
- Note added 2026-02-05
