# Security Policy

## Reporting a problem

This repository is written notes, not running code, so there's no software here to
exploit. Two things are still worth reporting privately:

**Dangerous guidance.** A runbook that tells an operator to do the wrong thing at 3am
is a real defect. If a command here would destroy data, widen access, or take a
service down in a way the surrounding text doesn't warn about, please say so.

**Leaked specifics.** These notes are meant to be generic. If a real hostname,
account ID, internal domain, credential or customer name has slipped into a file,
report it privately rather than opening an issue that quotes it.

Report either through GitHub:
[**Report a vulnerability**](https://github.com/JIUNG9/sre-runbooks/security/advisories/new)

> **Maintainer note:** that link requires *Private vulnerability reporting* to be
> switched on under **Settings → Advanced Security**. It's one checkbox and it's
> free on public repos.

Ordinary corrections — a flag that changed, a deprecated API, a better approach — are
welcome as normal issues or pull requests. No need for a private channel.

## Scope

Every URL, hostname and identifier in these notes is a placeholder
(`{{service}}`, `github.com/org/module`, `etcd-0:2379`). They're illustrative and
resolve to nothing.
