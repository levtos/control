# Codex bootstrap

This is the canonical short Codex start bridge. Repository-local `AGENTS.md`
files link here; they do not copy the detailed governance in `control/docs/`.

The [working rules for agents](working-rules.md) are binding for autonomy,
self-correction (up to 3 attempts), hard stops, test scope, backups, Issue
format, and reports. Where this bootstrap is stricter, the working rules win.

1. Use CTX first when it is available in the local environment. If it is not
   available, record that fact and continue without treating the absence as
   technical evidence.
2. Classify the assignment: read-only evidence, decision clarification, or
   implementation of an already decided scope.
3. Read the complete GitHub Issue, all comments, labels, and Project fields.
4. Read ADR 0002, the relevant `control/docs/` pages, repository rules, and
   the matching specification before technical work.
5. Treat GitHub as the only active and historical work, evidence, and memory
   source.
6. Confirm that the Issue contains an **Aktueller verbindlicher Vertrag**
   section near the top. It is the current target and explicitly supersedes
   older intermediate states; historical comments remain audit evidence and
   are marked `superseded`/überholt when replaced.
7. For stateful logic, verify that acceptance criteria name states,
   transitions, owner domain/repository, authoritative source, and expected
   results.
8. Keep observation, evidenced Ist, decided Soll, implementation, tests,
   risks, open gates, and Live evidence separate.
9. Before implementation, write a short Ist-/Soll-Abgleich against the
   canonical contract. If the target is missing or ambiguous, stop and record
   the product question in the Issue.
10. Carry explicitly linked cross-thread Issues, PRs, specifications, and
    decisions into the active prompt and handoff. Work on
    [`benni_media_state#21`](https://github.com/Levtos/benni_media_state/issues/21)
    must explicitly include [`control#28`](https://github.com/Levtos/control/issues/28)
    and [control PR #39](https://github.com/Levtos/control/pull/39).
11. Never invent a product decision or silently replace uncertainty. Preserve
    `unknown`, `unavailable`, `stale`, `source_conflict`, and
    `fallback=reject` as visible diagnostic outcomes.
12. Keep domain priorities, blockers, recovery, and existing paths
    domain-specific; do not create a global replacement stack.
13. Work under exactly one Issue agent, recorded as `agent:codex` or
    `agent:claude`. Codex is the current default for new or explicitly
    reassigned work; Claude is an explicit per-Issue exception.
14. Technical completion and Tests Pass are not Live. Benni owns Live and
    Live Verified.
15. Keep local bridges short and repository-specific; do not duplicate
    detailed governance or feature logic in integration repositories.

## Mandatory HACS implementation closeout

For every decided implementation assignment in a Home Assistant/HACS
integration, the same assignment includes the complete technical chain:

1. run focused tests and review the scope and diff;
2. open the pull request, including the stable version bump required by the
   repository rules;
3. inspect the checks and merge the pull request server-side;
4. verify the resulting default-branch SHA;
5. create the matching stable `vX.Y.Z` tag through the repository's approved
   path and wait for the existing release automation;
6. verify the normal, non-draft, non-prerelease GitHub Release and the HACS
   visibility or equivalent release evidence;
7. hand the work over in `Testing` with commit, pull request, merge SHA,
   checks, release, HACS evidence, risks, and the remaining Live gate.

There is no separate follow-up assignment or additional Benni approval gate
between pull request, merge, tag, and release. An agent may stop before merge
or release only when the assignment is explicitly read-only, an audit,
planning, or decision clarification; a concrete technical or product blocker
exists; checks fail; Benni explicitly stops that assignment before merge or
release; or a repository rule requires an additional explicit approval. The
blocker and the incomplete steps must be recorded; general caution alone is
not a blocker.

Merge, tag, GitHub Release, and HACS visibility are technical prerequisites
for Benni's test. They are not a HACS installation, Home Assistant deployment,
reload, restart, `Live`, or `Live Verified`. Benni owns those later gates.

This rule implements the original process decision in
[control#15](https://github.com/Levtos/control/issues/15) and its bootstrap
clarification in [control#50](https://github.com/Levtos/control/issues/50).

The Issue records the current scope, decisions, tests, PRs, risks, handoff, and
remaining Live gate. Benni owns the final Live verification.
