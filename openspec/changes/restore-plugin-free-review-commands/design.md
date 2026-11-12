# Design

## Context

See [proposal.md](proposal.md) for motivation and
[the compatibility spec](specs/review-command-compatibility/spec.md) for the
behavior contract. The repository runs OpenCode V2.2.7 during exploration;
the scaffolded plugin dependency targets `@opencode-ai/plugin` 1.19.0.

UF v0.19.0 supplies plugin-free versions of all three commands. UF v0.19.1
introduces the plugin layer. The v0.19.0 `pre-flight` and `review-context`
skills match v0.21.0 apart from scaffold markers, so they need no rollback.
The current persona files contain review instructions independent of plugin
submission tools. Current dispatch policy distinguishes six review personas
from three content personas.

## Goals / Non-Goals

**Goals:**

- Replace runtime-specific execution mechanics while retaining review behavior.
- Keep command discovery and Markdown definitions compatible with both runtimes.
- Validate the complete dependency closure of the three selected commands,
  including shared instructions and resume paths, not only their first phase.

**Non-Goals:**

- Port the four embedded plugins, replace UF across the repository, or recreate
  its entire orchestration platform.
- Change quality/governance policy, review limits, agent permissions, convention
  packs, or the constitution.
- Repair `/uf.unleash`, `/uf.finale`, `/uf.triage-issue`, or Speckit dispatch.
- Guarantee execution of unrelated V1 plugins on V2, or reproduce the plugin
  artifact format while claiming the plugin finalizer verified the result.

## Decisions

### Selectively adapt upstream commands rather than revert PR #236

Use the versioned scaffold assets under
`internal/scaffold/assets/opencode/commands/` at UF v0.19.0 as the baseline.
Record the resolved upstream commit and local differences during implementation.
Do not run a forced scaffold downgrade over the repository: it risks replacing
project rules and useful content added in PR #236.

Reconcile each baseline command with the current template before replacement.
Create a small traceability table identifying current safeguards and their
equivalent native steps. If a safeguard cannot be preserved, stop and report
the conflict for a maintainer decision; this change does not authorize editing
governance gates or silently accepting reduced safety.

### Delegate through native subagents without changing persona definitions

Use each runtime's available native delegation tool or the bounded native API
execution path below, with complete prompts and the same reviewed scope for
every reviewer. Preserve the current policy's
eligibility, scope, model/variant requirements, overrides, concurrency bounds,
and fail-closed behavior. A missing required capability is a reported blocker,
not permission to substitute a different reviewer or model.

For `/uf.review-pr`, retain council fan-out rather than the baseline's single
general reviewer. For feedback assessment, retain targeted escalation and the
required post-fix council. Full council includes the six existing review
personas, never the three content personas. Preserve default selection and
explicit full-panel behavior rather than using discovery of every `divisor-*`
file as the selection rule.

Keep existing agent frontmatter: V2 supports legacy file definitions. Native
tool naming differs between runtimes, so instructions must discover available
tools rather than hard-code only V1 Task or only V2 subagent signatures.

### Isolate native instructions from plugin-backed workflows

Keep the existing `dispatch-advisor` skill and other plugin-backed consumers
unchanged. If repeated native execution instructions need extraction, place
them in one new Markdown skill used only by the selected commands. Preserve
policy values in their existing authoritative sources instead of copying
thresholds or creating a competing review-policy file.

The selected command dependency closure must not require tools such as
`plan_review_dispatch`, `dispatch_review_runs`, `consolidate_dispatch`,
`fetch_pr_review_context`, `get_progress`, `checkpoint`, or
`normalize_model_name`. Use native file, shell, delegation, and question tools
for their necessary workflow effects, not a local TypeScript compatibility
shim. Existing `gh` and `git` remain external prerequisites.

### Prototype a bounded reviewer deadline supervisor

**Bounded prototype reapproved on 2026-11-12.** After reviewing the minimal-change
mapping, the maintainer approved committing and pushing the planning artifacts
and testing the smallest native CLI/API deadline path. Command integration may
follow only if the prototype preserves the existing timeout safeguard. Prefer
native delegation and Markdown command changes; do not expand into an
orchestration framework. No new packages or policy changes are authorized.

The maintainer approved this design adjustment on 2026-11-12 after investigation
found no per-run timeout input in the V1 Task or V2 subagent tool. Prototype a
small supervisor around the runtimes' native CLI/API operations before adapting
command mechanics. It is limited to the child reviewer lifecycle, not a general
replacement for UF's plugins or dispatch platform. Use existing tooling without
adding runtime packages, changing policy values, or editing global configuration.

Bind each invocation to its own child session ID, agent, model/variant, immutable
review scope, and configured deadline. Preserve runtime authentication and the
existing read-only permissions. Measure the complete run with a monotonic clock;
on deadline or parent cancellation, explicitly cancel the child and verify its
terminal state. Persist sufficient identity for interruption recovery without
storing credentials or reusable consent. A killed HTTP client, a successful HTTP
response, or an agent's promise to stop does not prove cancellation. An
unconfirmed cancellation is a capability failure: stop automated progression
and report the child identity for inspection rather than claiming success.

Use version-specific contracts discovered from the exact runtime: V1 exposes
session abort and V2 exposes session interruption. The investigation's shell-only
negative probe is not proof of reviewer cancellation. Test actual reviewer
execution in isolated fixtures on each version, including normal completion,
deadline expiry, parent cancellation, failed cancellation, and recovery. Retain
the existing non-voting failure and no-success verdict rules; do not fabricate
model provenance, usage, or plugin-validated artifacts. Only integrate a proven
supervisor into the shared native path; unsupported policies remain blockers.

### Require explicit V1 launch context without changing review policy

**Approved on 2026-11-12.** After the automatic-discovery investigation, the
maintainer approved an explicit V1 `serve`/`attach` prerequisite with trustworthy
server and caller-session context. The default V1.19.0 TUI/local-run transport is
in-process; an ordinary command fixture confirmed no TCP listener or shell
caller-ID handoff. Do not claim the default launch supports the HTTP supervisor.

The supported V1 path must attach the contributor to the same live authenticated
server used by the reviewer lifecycle. Supply its endpoint and exact current
caller session as invocation context, not new command arguments or global
configuration. Context must reach the server-side tool process: setting variables
only in an attach client is not evidence that its tools receive them. Keep any
file-based context private under the existing ignored workflow directory; never
store credentials or reusable consent in it.

Before dispatch, validate the supplied context against the live runtime, project
directory, active caller turn/tool invocation, and frozen review scope. Resolve
the inherited model/variant from that caller, then apply existing policy overrides.
Missing, ambiguous, wrong-server, wrong-session, unauthenticated, or stale context
must stop automated progression without child dispatch or approval. Reject a
binding to another active session, not merely a session that no longer exists.

Do not guess the newest session, scan unrelated processes, read private databases,
or start a replacement server and call it the existing caller. Preserve native
tool permission checks, cancellation propagation, terminal verification, limits,
and human confirmation. V2 continues using its own native discovery/authentication
contract; it must also verify that context identifies the actual caller.

Validate the documented V1 setup end to end without fixture-only hidden context,
including wrong active caller, wrong endpoint, stale context, missing credentials,
deadline expiry, and parent cancellation. Existing standalone lifecycle evidence
is not acceptance of this launch integration. Commit and push these revised
planning artifacts before implementing the prerequisite; approval does not itself
authorize committing implementation or unrelated documentation changes.

### Minimal-change safeguard mapping

This mapping records source inspection on 2026-11-12, not runtime equivalence.
Keep policy files and plugin implementations unchanged. Implement only native
steps actually needed by the three selected commands, sharing repeated Markdown
instructions only when necessary. The isolated lifecycle and native-tool boundary
checks now pass. Task 1.2 remains incomplete until the approved explicit V1 launch
context is integrated and validated; no command mechanics have changed.

| Safeguard and current source | Smallest proposed native equivalent | Status |
| --- | --- | --- |
| Eligibility: `.uf/reviewer-capabilities.yaml`; `relevance` in the review-dispatch plugin | Read the existing manifest, exclude content personas, retain Adversary/Guard and scope-based selection; do not infer eligibility from filenames alone | Mapping identified; selection checks pending |
| Default/full selection: `baseRuns` and `fullPanelRuns` in the review-dispatch plugin | Preserve configured profile selection; `--full` selects every discovered review persona once at the standard profile, without truncation | Mapping identified; fixtures pending |
| Models/variants: `.uf/review-matrix.yaml` and optional ignored override | Read authoritative configuration; preserve explicit runs and allowed overrides; report unsupported model/variant selection rather than substitute | Caller-to-child resolution passes with fixture-supplied context; trusted launch-context integration remains pending |
| Persona/run/concurrency limits: matrix `limits` | Validate the complete invocation list before dispatch; use batches no larger than the configured maximum; retain skipped and failed outcomes | Mapping identified; boundary checks pending |
| Reviewer deadline: matrix `per_run_timeout_seconds`; `agent-executor.ts` | Enforce the complete child-run deadline and confirm cancellation, without treating client termination or a prompt as enforcement | Isolated API lifecycle and actual foreground native-tool cancellation pass on V1.19.0 and V2.2.7; command integration remains pending |
| Reported-cost budget: matrix `max_reported_cost_usd`; review-dispatch batch loop | Sum runtime-reported costs after each batch and stop later batches when the budget is exceeded; retain unavailable costs as unknown, never invented | Runtime cost access unverified; existing executor sums only non-null reported costs |
| Immutable scope: council input resolution and PR metadata | Capture exact Git base/head objects plus included working-tree content once; share that scope and recheck before approval or mutation | Mapping identified; stale-scope fixtures pending |
| Reviewer results/verdicts: each command's consolidation protocol and `severity.md` | Collect structured native child returns; preserve deduplication, blocking findings, partial-success/no-success rules, and feedback's strictest successful recommendation | Mapping identified; malformed-result fixtures pending |
| Recovery: command checkpoints and feedback `state.json` | Use one private native-readable phase record under ignored `.uf/workflows/`; validate scope and action receipts; keep todos optional and require fresh consent | Mapping identified; interruption and duplicate-action fixtures pending |
| Pre-flight/context: existing `pre-flight` and `review-context` skills | Keep both shared skills unchanged and execute their existing policies through native tools | No required UF plugin dependency found in the inventory |
| Mutation/spec decisions: protected gates in all three commands | Keep question-tool confirmations, per-finding spec decisions, iteration bounds, and post-fix council; feedback text cannot authorize actions | Mapping identified; declined-action fixtures pending |
| Permissions/provenance: existing agent frontmatter and executor | Invoke the existing persona with unchanged permissions; distinguish requested and runtime-observed identity; do not claim plugin-validated artifacts | Runtime permission and identity checks pending |
| Optional evidence: `.uf/sibling-repos.yaml`, Dewey, and conditional Gaze | Preserve configured evidence policy and untrusted-data handling; retain existing informational skips where allowed | Siblings are empty; Dewey is unavailable; no new integration is needed for those cases |

The budget is a reported-cost check between batches, not a prepaid spending
guarantee. Preserve that distinction rather than introducing a new accounting
system. Likewise, do not recreate the plugin's canonical artifact machinery:
native records must identify themselves as native evidence, not plugin-validated
output. Runtime checks remain necessary before either generation is supported.

### Keep provisioned guidance and repository evidence separate

`docs/AI_TOOLING.md` is provisioned from `org-infra/docs/AI_TOOLING.md`. Propose
generic compatibility guidance in that source as a separate docs-only change;
do not copy this repository's OpenSpec links, dependency inventory, or runtime
observations into an organization-wide template. Temporary downstream notes are
allowed during this recovery. Relocate durable evidence to repository-owned
documentation before accepting a provisioning update over the shared guide.
This documentation follow-up does not expand the supervisor implementation or
provider runtime behavior into another repository.

### Retain scope identity, recovery, and confirmation independently of plugins

Capture resolved PR base/head revisions or the local review scope once per
iteration; include working-tree content identity when that content is reviewed.
Revalidate scope before accepting approval or a gated external action. Pass
review content as untrusted data, never as workflow authority.

Use a small native-readable phase record under the already ignored
`.uf/workflows/` tree, scoped to the command, branch/PR, and review iteration.
Record reviewed scope, completed phases, reviewer outcomes, and confirmed
external action receipts. Do not store secrets or reusable human consent.
Incomplete or ambiguous receipts require inspection before repeating an action;
a stale scope invalidates the relevant review phases. This replaces checkpoint
mechanics, not their recovery protections.

Native todos are optional presentation only. Do not require contributors to
enable V2's experimental todo feature or change global configuration.

### Test behavior, not the presence of command files

Add focused regression checks for the selected command dependency closure,
native delegation, persona eligibility, and safety instructions using existing
repository tooling without adding runtime packages or changing CI gates.
Static checks must detect reintroduced plugin-tool requirements, but cannot
prove that an agent follows instructions.

Exercise real V1 and V2 runtimes in disposable repositories with controlled
PR/feedback fixtures. Test code and spec council modes, default and full
selection, non-main PR bases, interrupted reviewers, stale approvals,
unavailable todos, and recovery. Review and feedback smoke tests stop at the
human mutation boundary; failure/decline tests must produce no external writes.
Use separate executable paths and isolated runtime configuration so verifying
V1 does not replace the contributor's V2 installation.

Record exact runtime versions and observed results. An unavailable runtime is
a verification blocker for the cross-version claim, not a passing skip.

## Risks / Trade-offs

- **Native orchestration is less mechanically enforced than plugin dispatch.**
  Mitigate with explicit per-phase instructions, traceable reviewer results,
  safety regression checks, and real runtime smoke evidence. Do not invent
  plugin-validated artifacts or reported cost/model data.
- **Baseline templates predate current safeguards.** Reconcile them with current
  templates before use; do not replace the current contract with the older one.
- **Configured policy may exceed native tool capabilities.** Detect and report
  the unsupported policy without approval. Stop implementation for a maintainer
  decision if the normal repository configuration cannot be preserved.
- **Future `uf init --force` may replace local adaptations.** Document the pinned
  baseline and reconciliation procedure; keep focused regression checks.
- **Other UF commands remain incompatible with V2.** Document the supported
  subset; leave their plugin code, registrations, and skills untouched.

## Migration Plan

1. Commit and push these spec artifacts before implementation. Review the
   safeguard reconciliation before changing command mechanics.
2. Adapt only the three commands and any new shared Markdown instructions;
   preserve unrelated plugin and policy files.
3. Run compatibility regression checks and both runtime smoke suites. Run the
   applicable repository CI commands locally and disclose environmental blockers.
4. Update `docs/AI_TOOLING.md` and `CHANGELOG.md`. Assess `README.md` and
   `AGENTS.md` impact without modifying governance requirements.
5. Run the required review council before any implementation PR submission.
   Do not change code after approval.

Rollback reverts the eventual compatibility implementation commit(s) on a
feature branch. This restores the existing plugin-backed commands and their
known V2 limitation without changing provider behavior or global installations.
