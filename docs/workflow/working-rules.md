# Working rules for agents

Binding for every Levtos repository and every agent (`agent:codex`,
`agent:claude`). Decided by Benni on 2026-10-24. Repository `AGENTS.md` and
`CLAUDE.md` files link here instead of copying these rules. An assignment only
names what deviates from them.

## Goal and autonomy

The goal of an implementation assignment is the finished result named in the
Issue, normally a merged PR and, where the Issue says so, a published release
that is installed. The agent decides on its own what is needed to get there and
does it without asking, including:

- installing, updating, or pinning dependencies, lockfiles, security fixes,
  GitHub Actions, and tooling (major version jumps only when no smaller update
  solves the problem);
- making flaky tests deterministic and repairing CI or workflow problems;
- fixing small defects found on the way, as long as they belong to the Issue's
  topic and do not change product behaviour;
- merging its own PR once the required checks are green, when the Issue
  includes merge or release.

Everything the agent fixed on its own is listed in the report (what, why,
evidence).

## Self-correction

When a step fails, the agent analyses the cause, applies a correction, and
tries again: at most **3 attempts per failing step**, each with a **new**
correction. Re-running without a change is allowed only for an evident runner
or network disturbance and counts as an attempt. After the third failed
attempt the agent stops and posts its analysis on the Issue.

## Hard stops

The agent stops instead of working around these, and records the reason on the
Issue:

- Deleting, skipping, or loosening tests, checks, or thresholds (skip markers,
  weaker assertions, raised timeouts as the only measure, lower audit level).
  Changing a test is allowed only with evidence that the test itself is wrong;
  the evidence goes into the report.
- Changing repository settings, branch protection, required checks, or secrets.
- Changing product behaviour of contracts, fusions, freshness rules, state
  machines, or other truth logic beyond the decided scope.
- Architecture or ADR decisions, versioning schemes, or widening the scope.
- Acting on a live installation beyond the assignment: no direct database
  access, no changes to Home Assistant, MQTT, Zigbee2MQTT, other apps, or
  legacy integrations; never deleting data.
- Printing or storing secrets (including tool calls that return app options
  with secrets), committing real media titles or private data, switching the
  GitHub account, or force-pushing a default branch.

## Size of an assignment

The Issue states the size.

- **Small** (fix, dependencies, tests, CI, documentation): build, green CI,
  merge, and release in one pass. No separate review or Go.
- **Large** (new function, database migration, change to truth logic):
  design PR → Claude review → Go → build → Claude review → Go → release and
  installation.

## Tests and checks

Locally only focused tests for the changed parts plus lint, format, and type
checks. The full suite runs in CI; the agent reads the result instead of
repeating it locally. A release run does not need additional local testing.

## Backups

Agents do not create Home Assistant backups before installing an app update;
those backups do not contain external databases. If a release migrates a
database, the Issue says whether Benni's own database backup is a
precondition, and the agent checks the Issue for it before installing.

## Issue as assignment

Benni starts work with one line, for example "Bitte bearbeite
Levtos/wuschel#42". Every assignment Issue contains:

1. **Ziel** – the result.
2. **Ausgangslage** – the relevant actual state with links.
3. **Umfang** – what belongs to it and what does not.
4. **Fertig, wenn …** – verifiable completion criteria.
5. **Größe** – small or large.

Assignment Issues may be written by Benni or by Claude (`levtos-claude`).

## Report and Go

- The agent posts its report as a comment on the Issue: at most one screen,
  with result, links (PR, run, release), the section **selbst behoben**, and
  open points.
- Claude posts reviews as PR comments.
- Benni gives a Go as an Issue comment containing "Go". The next instruction to
  the agent can then be "Issue #… hat Go, bitte fortsetzen."

## Unchanged

GitHub identity preflight, role boundaries, and privacy rules in
[agent orchestration](agent-orchestration.md) remain binding. Technical
completion is not Live; Benni owns the Live gate.
