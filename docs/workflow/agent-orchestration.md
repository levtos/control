# Agent orchestration

This document is the binding detail rule for separating product architecture,
technical evidence, and implementation in the GitHub-first workflow. It
complements the operating model, transparency, verification, and release
documents linked from this directory.

## Authority and role boundaries

Benni is the product decision maker and architect. He decides desired
behaviour, priorities, scope, and the final live verification gate.

ChatGPT clarifies the problem with Benni, reads the responsible GitHub Issue and
its linked documentation, separates evidenced actual state from hypotheses and
desired state, evaluates technical evidence, and writes the complete
implementation assignment after the product decision is clear. In the project
folder ChatGPT is read/review/assignment-only: it does not make code,
configuration, test, workflow, deployment, or live-system changes.

Codex and Claude provide technical evidence and implement an already decided
scope. They may identify contradictions, risks, or unknown dependencies, but do
not decide the desired product state independently. A technical unknown is
handled as a read-only evidence assignment, not as an implicit architecture
decision.

Exactly one of the two agents owns implementation for an Issue, recorded as
`agent:codex` or `agent:claude`. This is per-Issue ownership, not a permanent
repository lock; both agents may work in any active repository when assigned.

## GitHub identity preflight

The binding GitHub identity mapping is:

| Role | GitHub account |
| --- | --- |
| Benni | `Levtos-Acc` |
| `agent:codex` | `levtos-codex` |
| `agent:claude` | `levtos-claude` |

Before every writing GitHub action, the acting agent must run
`gh api user --jq .login` and verify that the result exactly matches the account
assigned to that agent. This applies to every write, including pushes, Issue
and Project updates, PR creation or updates, merges, workflow dispatches, tags,
and releases; an earlier session check does not replace this preflight.

If the actor does not match, the agent must not perform the write, must not run
`gh auth switch`, and must not fall back to another account. Report the identity
mismatch as a blocker. If the check fails or the actor cannot be verified, the
write is blocked as well.

Commit author and GitHub actor remain distinct evidence. Record commit author,
push actor, PR actor, merge actor, workflow actor, and release actor separately;
a commit author line does not prove which GitHub account performed an action.
Never include tokens or other credentials in this evidence.

## Assignment types

### Read-only evidence

The agent reads the complete Issue and relevant documentation, inspects only the
named code paths, contracts, dependencies, logs, and live evidence, and reports
files, functions, observed behaviour, cross-repository dependencies, and
uncertainties. It changes nothing and does not propose a product decision as if
it were evidence.

### End-to-end implementation

The agent implements the decided scope minimally in an isolated branch,
inspects necessary cross-repository paths, checks permissions and automation
early, adds risk-appropriate tests, opens a PR, responds to review findings,
merges server-side when authorized, verifies the resulting default-branch SHA,
and records the result on the Issue. Mandatory scope-conforming follow-up work
is part of the assignment. Stop only for a new product decision, material risk,
or a blocker that cannot be resolved safely.

### Handoff

A handoff is a single explicit Issue update containing current Soll/Ist,
branch/commit, changed files, tests, risks, open decisions, and the next step.
The outgoing agent stops implementation before the incoming agent starts; the
incoming agent verifies the state and takes the ownership label. There are no
parallel implementations or competing solution branches for one Issue.

## Risk-based review

A second review is useful for cross-repository contracts, large or unexpected
diffs, safety-critical changes, unclear side effects, or concrete doubts. The
reviewer remains read-only and reports all findings together. Agents do not
enter review-and-fix ping-pong or build parallel solutions without an explicit
comparison assignment.

## Issue contract

The responsible GitHub Issue is the durable work context. Keep these concerns
separate:

- observations, timestamps, logs, and individual episodes;
- evidenced technical actual state;
- Benni's decided functional target state;
- implementation and verification evidence;
- remaining decisions, risks, and blockers;
- explicit documentation impact: no change, existing page updated, or new page.

Implementation starts only when no product decision remains open. Use the
GitHub Issue and the Platform Workflow Project for current work; do not create
an alternate board or use retired platforms.

## Project Memory

Stable, verified, generally reusable knowledge belongs in the versioned
documentation under docs. Concrete incidents, raw logs, hypotheses, rejected
variants, and historical intermediate states remain on the Issue. Planned or
unconfirmed behaviour must not be presented as live architecture.

Before new architecture discussion, read the responsible Issue and relevant
contracts, ADRs, Lastenhefte, and workflow pages. Before completion, state the
documentation impact explicitly.

## GitHub, releases, and live gates

GitHub is the only active and historical workflow, evidence, Project Memory,
and public distribution source. GitLab, Plane, and Forgejo are retired and are
not valid alternatives, archives, or rollback targets; migration links are
non-authoritative provenance only.

Repository-local tests and central GitHub release automation are separate
layers. For HACS repositories, use the established GitHub Action and stable
version/tag flow. A successful PR merge or release Action does not equal live
Home Assistant acceptance: keep technical completion separate from Benni's
live gate.

## Privacy and scope

Never commit secrets, private MCP URLs, access tokens, private Home Assistant
configuration, or internal topology to a public repository. Sanitize historical
imports and retain sensitive archives only in an explicitly protected,
recoverable location. Keep work scoped to the Issue; do not add unrelated
cleanup or broad audits.

## References

- operating-model.md — target and actual state, roles, and gates.
- transparency-rules.md — Issue-first work and durable notes.
- verification-and-efficiency.md — minimum verification by risk.
- hacs-release-workflow.md — stable release path.
- ../migration/issue-mapping.md — historical-to-current Issue mapping.
- ../adr/0002-github-only-governance.md — binding governance decision.
- ../operations/repositories.md — lifecycle statuses and registry.
