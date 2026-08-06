# Contributing

Corrections are the most useful thing you can send. These notes go stale in specific,
predictable ways: a CLI flag gets renamed, an API version is deprecated, a default
changes, or the recommended approach is simply better now. If you hit one, open an
issue or a PR — a one-line fix is welcome and doesn't need justification.

## What fits here

Short, specific, and learned from something that actually happened. The test is whether
a colleague mid-incident would find the answer faster here than in the official docs:
the parameter that turned out to matter, the failure mode that wasn't obvious, the
command that actually diagnosed it.

What doesn't fit is a tutorial. Nothing here explains what Kubernetes is, and the notes
assume you already know the tool and need one specific thing from it.

New topics are fine, but a file with a heading and nothing under it isn't. This repo
previously carried 284 lines of exactly that — sections whose entire body was a date —
and they were removed because an empty section costs a reader time and tells them
nothing.

## Placeholders, not real values

Every hostname, account ID and URL in these notes is deliberately fake:
`{{service}}`, `github.com/org/module`, `etcd-0:2379`, `db01.prod.internal`. Keep it
that way. If you're adapting something from a real environment, strip the identifiers
before it lands here.

## Before you open a PR

```bash
npx markdownlint-cli2 "**/*.md"
```

That's what CI runs, and it's the whole gate. `--fix` handles the spacing rules
automatically.

Keep the existing shape: an `#` title, then `##` sections. For a new runbook,
[`sre/runbook-template.md`](sre/runbook-template.md) is the structure to copy —
overview, diagnosis, mitigation, escalation.

## One exception to "just open an issue"

If the problem is that a note here would be **dangerous to follow** — it destroys data,
widens access, or takes a service down without the text warning you — read
[SECURITY.md](SECURITY.md) first. That goes through a private channel rather than a
public issue that quotes the bad advice.
