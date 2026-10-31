# Game Day Template

A game day is a planned failure injected into a system that people are watching. It
tests two things: whether the system degrades the way you believe it does, and whether
the people and runbooks respond the way you believe they do. Fill this in at least a
week before the date and get the abort criteria agreed in writing.

## Scope

- **System under test:** {{service}} in {{environment}}
- **Environment:** staging first. Run in production only after the same exercise has
  passed in staging twice.
- **Date, window and duration:** {{date}}, {{start}}-{{end}}, hard stop at {{end}}
- **Out of scope:** list the dependencies you will not touch, and the teams that
  share them.

## Hypothesis

State it as a prediction with numbers, so the result can be wrong.

```text
When {{failure}} is injected for {{duration}},
{{SLI}} stays above {{threshold}} for {{percent}}% of requests,
and {{alert}} fires within {{minutes}} minutes.
```

A hypothesis without a threshold ("the service stays up") cannot fail, so it teaches
nothing.

## Blast radius

| Question | Answer |
| --- | --- |
| Which users or tenants can see the impact? | |
| Which other services call the target? | |
| What is the maximum traffic share affected? | |
| Does the failure touch shared infrastructure (node pool, database, queue)? | |
| Is there a customer commitment (SLA, launch, billing run) in the window? | |

Limit the injection to the smallest unit that still tests the hypothesis: one node
instead of a node group, one replica instead of a deployment, 10% of requests
instead of all.

## Abort criteria

Write these as measurable conditions with one person who can call the abort. Agree on
them before the day, because nobody judges well mid-incident.

| Condition | Action |
| --- | --- |
| {{SLI}} below {{abort threshold}} for {{minutes}} minutes | Stop the injection and roll back |
| Any alert outside the expected set fires | Pause and have the incident commander decide |
| A customer-facing error report arrives | Stop the injection |
| The rollback step does not restore state within {{minutes}} minutes | Declare a real incident and use the normal process |

Every injection needs a tested rollback command written in this document. If you
cannot write the rollback, do not run the injection.

## Roles

| Role | Does | Name |
| --- | --- | --- |
| Game master | Injects the failure, owns the timeline and the abort call | |
| Incident commander | Runs the response as in a real incident, without knowing the injection details | |
| Responders | Diagnose and mitigate using the runbooks only | |
| Observer | Records timestamps and decisions, does not help | |
| Comms | Posts status to the channel, as in a real incident | |

If the responders need to know the injection to avoid harm, tell them. Otherwise keep
the details from them, because the purpose is to find out how they diagnose it.

## Timeline

Record these timestamps during the run. They become the measurements.

| Event | Time |
| --- | --- |
| Injection started | |
| First alert fired | |
| First human acknowledged | |
| Cause identified | |
| Mitigation applied | |
| SLI back within threshold | |
| Injection rolled back | |

## Results

- **Hypothesis held:** yes / no, with the measured values
- **Time to detect, time to mitigate:** from the timeline above
- **Runbook gaps:** steps that were missing, wrong or out of date
- **Surprises:** behavior nobody predicted

## Action items

Each item has an owner and a date. Items with neither are not tracked.

| Item | Owner | Due |
| --- | --- | --- |
| | | |

Repeat the exercise after the fixes land. A game day that finds a gap and is not
re-run does not show the gap is closed.
