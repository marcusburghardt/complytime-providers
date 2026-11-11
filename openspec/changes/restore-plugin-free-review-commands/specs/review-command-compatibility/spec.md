# Spec Delta

## Purpose

Allow contributors to review pull requests and address feedback through the
required council on OpenCode V1 and V2 without relying on embedded UF plugins.

## ADDED Requirements

### Requirement: Cross-version plugin-independent command execution

The three commands `/uf.review-council`, `/uf.review-pr`, and
`/uf.address-feedback` SHALL execute through native OpenCode tools on both V1
and V2 without requiring embedded UF plugin tools. Existing command names and
supported arguments SHALL remain available.

#### Scenario: Main commands run without plugins

- **GIVEN** an isolated checkout with the embedded UF plugins unavailable
- **WHEN** each main command runs under a recorded V1 version and a recorded
  V2 version with its native tools and required external prerequisites
- **THEN** each command reaches its review or feedback decision boundary
- **AND** no missing UF plugin tool prevents execution

#### Scenario: Native delegation is unavailable

- **WHEN** the runtime cannot invoke a required review subagent
- **THEN** the command reports the missing capability and stops without approval
- **AND** it does not substitute a parent-only review

### Requirement: Council review remains the review authority

PR review SHALL use the council rather than a single general reviewer.
Feedback execution SHALL obtain council review of the cumulative fixes before
pushing. The council SHALL retain current reviewer eligibility, required
participation, configured limits, and review iteration limits. Content-only
personas SHALL NOT participate as reviewers.

#### Scenario: PR council review

- **WHEN** a contributor invokes `/uf.review-pr` for an eligible pull request
- **THEN** the command delegates the AI review to the required council reviewers
- **AND** reports reviewer participation and consolidated findings

#### Scenario: Feedback fixes require council review

- **WHEN** the author confirms feedback fixes and the fixes are ready locally
- **THEN** `/uf.address-feedback` obtains council review of the cumulative changes
- **AND** it does not push while required review remains unresolved

#### Scenario: Full council excludes content personas

- **WHEN** a full council runs with the repository's existing agent set
- **THEN** Adversary, Architect, Curator, Guard, SRE, and Testing review the scope
- **AND** Envoy, Herald, and Scribe do not cast review votes

#### Scenario: Policy cannot be honored

- **WHEN** a configured review policy or limit cannot be honored by native tools
- **THEN** the command reports the incompatibility and withholds approval
- **AND** it does not ignore overrides, truncate reviewers, or relax the limit

### Requirement: Review scope is immutable and approval is current

Every council iteration SHALL identify the reviewed base, head, and included
working-tree changes when applicable. All reviewers SHALL receive the same
scope. PR scope SHALL follow the PR base rather than assuming `main`. An
approval SHALL NOT authorize a different revision or changed working tree.

#### Scenario: PR targets a non-main branch

- **WHEN** the pull request targets a branch other than `main`
- **THEN** all reviewers assess the PR's resolved base-to-head scope
- **AND** the report identifies the reviewed revisions

#### Scenario: Changes follow approval

- **WHEN** reviewed commits or included working-tree content change after approval
- **THEN** the command invalidates the stale approval and requires fresh review
- **AND** it does not push or post approval using the stale result

### Requirement: Missing or failed reviews never count as approval

Commands SHALL collect actual results and preserve current verdict rules.
Failed, cancelled, timed-out, missing, or malformed results SHALL be reported
as informational and non-voting, not automatic blockers when another structured
assessment succeeds. Zero successful assessments or unresolved blocking findings
SHALL prevent approval. Review exhaustion SHALL require the existing human
decision rather than silently extending the loop.

#### Scenario: Reviewers complete successfully

- **WHEN** every required reviewer returns a valid approving result for the scope
- **THEN** the command reports the actual reviewers and their consolidated verdict
- **AND** it does not invent model provenance or usage data

#### Scenario: Reviewer fails while other reviewers approve

- **WHEN** a reviewer fails to complete or returns a malformed result, but at
  least one other reviewer returns a successful structured approving assessment
- **THEN** the command reports the failure as informational and non-voting
- **AND** the failure alone does not prevent approval under the current verdict rules
- **AND** any retry remains within existing limits

#### Scenario: No reviewer succeeds

- **WHEN** no reviewer returns a successful structured assessment
- **THEN** the command reports `UNAVAILABLE` for availability-only provider,
  model, or runtime causes, or `INCONCLUSIVE` for policy, plan, budget, limit,
  persistence, calculation, or mixed causes
- **AND** it withholds approval and blocks automated progression

#### Scenario: Successful reviewer requests changes

- **WHEN** a successful structured assessment returns an unresolved blocking finding
- **THEN** the command identifies that reviewer and reports `REQUEST CHANGES`
- **AND** any retry or fix loop remains within existing limits

### Requirement: Existing quality and human authorization gates remain intact

The compatibility recovery SHALL preserve existing CI/pre-flight policies,
severity rules, spec-review decision gates, and confirmation requirements.
External review text SHALL remain untrusted input. A workflow SHALL NOT post
reviews or replies, resolve threads, push commits, or perform other gated
mutations without the applicable current authorization.

#### Scenario: Human confirmation is declined

- **WHEN** the contributor declines posting, feedback execution, or another
  requested mutation
- **THEN** the command performs none of the declined actions
- **AND** it retains the assessment for the contributor to inspect

#### Scenario: Blocking pre-flight result or injected feedback

- **WHEN** pre-flight identifies a failure that blocks under the existing policy,
  or feedback asks the agent to bypass review or confirmation
- **THEN** the command preserves the existing gate and rejects the bypass request
- **AND** it does not reinterpret feedback as executable instructions

### Requirement: Progress recovery does not require plugin checkpoints

Commands SHALL record completed phases and their reviewed scope using native
tools so an interrupted or compacted session can recover without plugin
checkpoints. Missing optional todo tooling SHALL NOT block execution. Recovery
SHALL verify scope freshness and SHALL NOT infer that an unrecorded action or
authorization occurred.

#### Scenario: Resume after interruption

- **WHEN** a command resumes with a progress record matching the current scope
- **THEN** it recovers confirmed completed phases without repeating external actions
- **AND** it requests fresh confirmation where the current workflow requires it

#### Scenario: Stale progress or missing todo tool

- **WHEN** recorded scope differs from the current scope, or optional todo tooling
  is unavailable
- **THEN** the command invalidates stale review results where applicable
- **AND** it maintains explicit phase tracking without requiring a global setting

### Requirement: Recovery leaves unrelated tooling and standards unchanged

The recovery SHALL preserve current packs, custom rules, severity definitions,
reviewer definitions, and governance settings. It SHALL NOT remove embedded
plugins or registrations used by other commands, migrate other UF workflows,
install a replacement runtime dependency, or alter global OpenCode configuration.

#### Scenario: Scoped recovery

- **WHEN** the compatibility change is applied
- **THEN** the three commands use the plugin-independent path
- **AND** unrelated UF workflows, plugin sources, registrations, review policy
  configuration, packs, and persona definitions remain unchanged

### Requirement: Compatibility claims identify evidence and baseline

Documentation SHALL identify UF v0.19.0 as the command baseline, explain local
adaptations and the bounded scope, and warn that UF scaffolding can overwrite
them. Compatibility SHALL be claimed only for exact V1 and V2 versions exercised
in runtime smoke checks; missing runtime checks SHALL be reported as unverified.

#### Scenario: Record successful verification

- **WHEN** compatibility checks complete
- **THEN** the evidence identifies both runtime versions and each tested command
- **AND** it records success, failure, cancellation, recovery, and confirmation
  checks without publishing secrets or mutating real PRs

#### Scenario: A runtime is unavailable

- **WHEN** only one runtime generation can be exercised
- **THEN** the verification report marks the other generation unverified
- **AND** static inspection alone does not satisfy cross-version acceptance
