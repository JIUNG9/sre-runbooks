# sre-runbooks

Reference notes for SRE / DevOps / platform work — the things worth writing down after
you've debugged them once.

[![CI](https://github.com/JIUNG9/sre-runbooks/actions/workflows/ci.yml/badge.svg)](https://github.com/JIUNG9/sre-runbooks/actions/workflows/ci.yml)
[![license](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![notes](https://img.shields.io/badge/notes-40%20files-1a8917)](#whats-here)

## What this is, and what it isn't

**It is** a set of terse, working notes: the command that actually diagnosed the
problem, the parameter that turned out to matter, the failure mode that wasn't
obvious. Written for someone who already knows the tool and needs the specific thing.

**It isn't** tutorials, and it isn't comprehensive. Each file is short on purpose. If a
topic isn't here it's because I haven't hit it in production, not because it's
unimportant.

Every hostname and URL in these notes is a placeholder — `{{service}}`,
`github.com/org/module`, `etcd-0:2379`. Nothing resolves anywhere.

## What's here

| Directory | Notes on |
| --- | --- |
| `kubernetes/` | DNS troubleshooting (ndots, CoreDNS, NodeLocalDNS), OOMKilled vs eviction, network policies, HPA tuning, RBAC |
| `terraform/` | State management and isolation, module patterns, `moved` blocks, import workflow, drift detection, testing |
| `sre/` | SLO framework, error budgets, observability stack, incident management, capacity planning, toil reduction, a runbook template |
| `aws/` | Cost optimization, IAM practices, RDS maintenance, Transit Gateway, secrets management |
| `cicd/` | GitHub Actions patterns, ArgoCD patterns, pipeline security |
| `postgres/` | Query optimization, vacuum tuning, replication, connection pooling |
| `docker/` | Multi-stage builds, security scanning |
| `linux/` | Performance debugging, systemd patterns, disk and inode exhaustion |
| `game-days/` | A game-day template (scope, hypothesis, blast radius, abort criteria, roles) and a worked node-drain exercise on EKS |

Start with [`sre/runbook-template.md`](sre/runbook-template.md) if you want the shape
I use for a new runbook: overview, diagnosis, mitigation, escalation.

## Contributing

Corrections are genuinely welcome — a flag that changed, an API that was deprecated,
or a better approach than the one written down. Open an issue or a PR.

If you find guidance here that would actually be *dangerous* to follow — destroys data,
widens access, takes a service down without warning — please read
[SECURITY.md](SECURITY.md) first; that one goes through a private channel.

CI runs markdownlint on every file. `npx markdownlint-cli2 "**/*.md"` reproduces it.

## History

Renamed from `sre-lab` on 2026-08-04 — "lab" suggested runnable code, and this is
written notes. GitHub redirects the old URL, so existing links still work.

## License

MIT — see [LICENSE](LICENSE).
