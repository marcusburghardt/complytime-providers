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

Use each runtime's available native delegation tool, with complete prompts and
the same reviewed scope for every reviewer. Preserve the current policy's
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
