# GitHub workflow

## Standard flow

1. Create or reuse a GitHub Issue.
2. Add it to the `Platform Workflow` Project and set known fields.
3. Set exactly one Issue implementation agent: `agent:codex` or `agent:claude`.
4. Read the Issue, comments, applicable Lastenheft, and repository rules.
5. Run the capability preflight.
6. Work in a clean isolated branch.
7. Run repository-local tests.
8. Open a pull request and inspect checks.
9. Merge server-side and verify the merge SHA.
10. For HACS work, create a stable patch tag and let the release Action create
   the normal release.
11. Verify HACS visibility. Leave the Issue at Testing until Benni's live
     gate.

Repository-local tests and central release automation are separate layers.
Do not create a test, runner, or CI gate that is not required by the Issue.

For decided Home Assistant/HACS integration implementations, steps 7 through
11 are one mandatory technical closeout in the same assignment. An agent may
stop before merge or release only for the documented exceptions in the
[Codex bootstrap](codex-bootstrap.md#mandatory-hacs-implementation-closeout)
and [stable HACS release workflow](hacs-release-workflow.md). Merge and release
remain separate from installation, deployment, `Live`, and `Live Verified`.
The original process decision is recorded in
[control#15](https://github.com/Levtos/control/issues/15).

## Documents

- [Working rules for agents](working-rules.md) — autonomy, retries, hard stops, Issue format
- [Operating model](operating-model.md)
- [Transparency rules](transparency-rules.md)
- [Project Memory](project-memory.md)
- [Stable HACS releases](hacs-release-workflow.md)
- [Verification](verification-and-efficiency.md)
- [Agent orchestration](agent-orchestration.md)
- [Codex bootstrap](codex-bootstrap.md)
- [ADR 0002: GitHub-only governance](../adr/0002-github-only-governance.md)
- [Repository registry and lifecycle](../operations/repositories.md)
- [GitLab retirement checklist](../operations/gitlab-retirement.md)
