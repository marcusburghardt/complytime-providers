# Tasks

Implementation starts only after these spec artifacts are committed and pushed.
All tasks below are implementation or implementation-verification work; creating
this proposal does not complete them.

## 1. Baseline and safeguard reconciliation

- [ ] 1.1 Resolve UF v0.19.0 to its upstream commit and inventory the selected
  commands' direct, shared-skill, and resume dependencies; verify the delivered
  inventory distinguishes native prerequisites from every required plugin tool.
- [ ] 1.2 Map current eligibility, model policy, limits, verdicts, immutable scope,
  progress recovery, pre-flight, and mutation gates to equivalent native steps;
  verify no governance change is needed and report any conflict before editing.
- [ ] 1.3 Document the baseline, safeguard mapping, supported command subset,
  overwrite warning, and isolated V1/V2 verification procedure in
  `docs/AI_TOOLING.md`; verify links and executable paths without replacing the
  contributor's installed runtime.

## 2. Native council execution

- [ ] 2.1 Adapt `/uf.review-council` and add a shared Markdown skill only where
  needed for native delegation and persistent phase recovery; verify code/spec
  modes, current default selection, full-panel selection, and native tool discovery
  have no required plugin-tool calls in their dependency closure.
- [ ] 2.2 Add focused regression checks for required review participation, content
  persona exclusion, policy/limit incompatibility, failed or malformed reviewer
  results, and stale scope; verify positive fixtures pass and negative fixtures
  cannot yield approval without adding dependencies or changing CI gates.
- [ ] 2.3 Validate native phase tracking with optional todos absent and an
  interrupted/compacted session; verify a matching record recovers phases, stale
  results are invalidated, and ambiguous action receipts do not trigger repeats.

## 3. PR review and feedback integration

- [ ] 3.1 Adapt `/uf.review-pr` to the shared native council path, retaining PR
  argument handling, actual base/head scope, CI-aware checks, and confirmation;
  verify a non-main-base fixture and a declined-posting fixture without writes.
- [ ] 3.2 Adapt `/uf.address-feedback` to plugin-independent ingestion, targeted
  escalation, author decisions, and council review of cumulative fixes; verify
  accepted/rejected feedback paths and that failed council review blocks push.
- [ ] 3.3 Extend regression checks across all three command dependency closures
  and document the recovered behavior; verify injection-like feedback cannot
  authorize mutations and no mandatory plugin or todo call remains.
- [ ] 3.4 Add the compatibility entry to `CHANGELOG.md` and assess `README.md`
  and `AGENTS.md` documentation impact; verify the diff preserves all packs,
  persona definitions, policy/configuration files, plugins, and unrelated commands.

## 4. Cross-version and repository verification

- [ ] 4.1 Run the three commands in disposable fixtures under an exact recorded
  OpenCode V1 version with UF plugins unavailable; verify successful reviews,
  failed/cancelled reviewers, stale approvals, recovery, and declined mutation
  gates, stopping normal feedback execution before external writes.
- [ ] 4.2 Repeat the same runtime checks under an exact recorded OpenCode V2
  version; deliver a per-command evidence matrix with results and redacted
  observations, treating an unavailable runtime as unverified rather than passed.
- [ ] 4.3 Run `openspec validate restore-plugin-free-review-commands --strict`,
  focused compatibility regression checks, and applicable local CI commands
  derived from `.github/workflows/`; verify build/test results and explicitly
  report any cross-repo integration or tool availability blockers.
- [ ] 4.4 Run `/uf.review-council` before submitting the implementation PR,
  resolve all required findings, and verify the approved scope remains unchanged
  through submission; retain the final review and runtime verification evidence.
