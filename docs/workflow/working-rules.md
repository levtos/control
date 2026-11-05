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

Autonomy is limited to the assigned scope. An unrelated pre-existing failure,
flaky check, or tooling defect is not a blanket permission to repair another
subsystem. First classify whether the failure was caused by the current diff;
if unrelated, record the evidence and do not broaden the scope. If causality
is uncertain, investigate narrowly and stop at the relevant decision boundary.
Required checks must never be bypassed to make a PR green.

## Context, token budget, and work packages

These efficiency rules apply to **all Levtos repositories** and to both Codex
and Claude. They optimize effort, not correctness or the technical closeout.

- Keep the Issue and release boundary intact, but divide large implementation
  work into bounded, independently verifiable packages. One package has a
  concrete change scope, expected files/interfaces, relevant tests, and a
  completion condition. Multiple packages may share one Issue and one beta;
  each may use a fresh agent session. Do not silently create additional Issues,
  parallel owners, releases, or extra approval gates.
- Start from the current binding Issue contract, linked decisions and last
  verified checkpoint. Read code and specifications **on demand** for the
  affected scope. Prefer targeted searches, symbol ranges and compact command
  output over repeated full-file dumps, repository-wide scans, massive logs or
  broad history rereads. This never excuses ignoring relevant dependencies,
  security requirements or stateful semantics.
- Keep tool output small: filter `gh` JSON to needed fields, inspect only
  failing test details, and avoid repeatedly fetching unchanged PR/CI data.
- A long session should consider a checkpoint and fresh session at about
  **40 model requests or 120,000 context tokens**, if that telemetry is
  available. These are **soft warning points**, not abort limits. Never abandon
  a half-written change, a required check, or a safety-critical action merely
  to meet them. If token counts are unavailable, do not waste tokens trying to
  calculate them; checkpoint at natural package or interruption boundaries.
- A checkpoint is short, durable and sufficient for continuation: Issue and
  package, branch and last verified commit, changed paths, completed work,
  tests/checks with their commit or run ID, failures and classification,
  unresolved risks/decisions, and the exact next action. Record it on the
  Issue or an explicitly linked versioned handoff; no secrets, private data,
  or large raw logs. A resumed agent verifies the current branch/commit and
  carries on from the checkpoint instead of starting a full new audit.
- Record actual context/token or budget use when the provider exposes it,
  without guessing billing from wall time or cached input tokens. Compare
  comparable small assignments over time; efficiency targets are not release
  or correctness gates.

## Self-correction

When a step fails, the agent analyses the cause, applies a correction, and
tries again: at most **3 attempts per failing step**, each with a **new**
correction. Without a code change (timeout, network disturbance, or a flaky
test), restart only failed CI jobs with `gh run rerun <id> --failed`, never the
whole run. The retry counts as an attempt. After the third failed
attempt the agent stops and posts its analysis on the Issue. A slip in the
agent's own command (typo, wrong variable or parameter name) whose fix is
evident is corrected immediately and does not count as an attempt; attempts
count only approaches that fail for a reason outside the agent's own slip.

## Hard stops

The agent stops instead of working around these, and records the reason on the
Issue:

- Deleting, skipping, or loosening tests, checks, or thresholds (skip markers,
  weaker assertions, raised timeouts as the only measure, lower audit level).
  Changing a test is allowed only with evidence that the test itself is wrong;
  the evidence goes into the report.
  Explicitly decided affected-path CI selection may omit unrelated jobs; it
  must not omit applicable tests or replace the full pre-merge release gate.
- Changing repository settings, branch protection, required checks, or secrets.
- Changing product behaviour of contracts, fusions, freshness rules, state
  machines, or other truth logic beyond the decided scope.
- Architecture or ADR decisions, versioning schemes, or widening the scope.
- Acting on a live installation beyond the assignment: no direct database
  access, no changes to Home Assistant, MQTT, Zigbee2MQTT, other apps, or
  legacy integrations; never deleting data.
- Printing or storing secrets (including tool calls that return app options
  with secrets), committing real media titles or private data, writing under
  any GitHub account other than the agent's own, or force-pushing a default
  branch. Switching **to** the agent's own account is allowed and expected
  (see [agent orchestration](agent-orchestration.md#github-identity-preflight)).

## Size of an assignment

The Issue states the size. A beta/release can remain one planned outcome while
its implementation is performed in smaller sequential packages and sessions.
This does not waive design reviews, explicit Go decisions, or the full final
release gate.

- **Small** (fix, dependencies, tests, CI, documentation): build, green CI,
  merge, and release in one pass. No separate review or Go.
- **Large** (new function, database migration, change to truth logic):
  design PR → Claude review → Go → build → Claude review → Go → release and
  installation.

## Tests and checks

Locally only focused tests for the changed parts plus lint, format, and type
checks. The full suite runs in CI; the agent reads the result instead of
repeating it locally. A release run does not need additional local testing.
After a correction, rerun only affected tests or the failed tests (`pytest --lf`
where appropriate). Do not repeat already green suites without a new change or
an unresolved concern. CI runs only jobs affected by the changed paths where
the repository provides a tested selector. Unknown paths fail closed; before
merging a release, the complete required set must pass.

Before running local tests, identify affected implementation paths, the direct
module tests and any contract/integration tests needed for changes to shared
interfaces, storage, quality, ordering, or restore behavior. Use a maintained
module/test mapping if available; otherwise select conservatively and report
why. UX-only changes do not trigger unrelated backend suites locally; a shared
contract change may require tests from multiple domains. Do not disable tests,
loosen assertions, or assume a passing selected set proves the full system.

Start CI once for the current commit and retain the run URL/ID. Avoid repeated
polling, `sleep` loops, and rereading entire successful logs inside a costly
agent session. Prefer a controlled handoff while CI is running when continued
waiting adds no useful work. Resume later to inspect the completed checks and
finish the **same** authorized merge/release chain. For an infrastructure or
flaky failure without a code change, retry only failed jobs as specified above;
record the result. A failed mandatory check remains a blocker until resolved
or explicitly decided; never merge around it.

## Release batching

Collect non-blocking corrections and publish them with the next regular release.
Create a separate fix release only when Benni is currently blocked. An explicit
combined release assignment decides the release boundary. This rule does not
remove the mandatory technical release chain or Benni's separate Live gate.

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
