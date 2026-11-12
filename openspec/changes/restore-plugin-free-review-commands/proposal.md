# Proposal

## Why

The UF v0.21.0 update in PR #236 made the main review commands depend on
embedded OpenCode V1 plugins that do not run on OpenCode V2. Restore these
commands on both runtime generations without maintaining a local plugin fork
or weakening the repository's review requirements.

## What Changes

- Recover the plugin-free workflow structure from UF v0.19.0 for
  `/uf.review-council`, `/uf.review-pr`, and `/uf.address-feedback`.
- Use native OpenCode subagent delegation. Retain council review for PR review
  and the post-fix feedback gate rather than restoring v0.19.0's single-agent
  PR review behavior.
- Preserve current convention packs, custom rules, severity definitions,
  review-persona definitions, and governance requirements. Exclude content
  personas from review dispatch.
- Replace plugin-specific mechanics within these commands with documented,
  cross-version native-tool instructions, including safe progress recovery,
  immutable review scope, result collection, and human confirmation.
- Require explicit authenticated `serve`/`attach` context for the supported V1
  path, as approved on 2026-11-12. Bind the exact current caller session without
  changing command arguments, global settings, or existing review safeguards.
- Document the v0.19.0 baseline, local adaptations, verified runtime versions,
  and protection against accidental replacement during later UF scaffolding.

This is one user story: a contributor can review a PR and address its feedback
through the required council on either OpenCode V1 or V2. Plugin migration,
wholesale UF downgrade/removal, provider changes, and restoration of other UF
commands are outside scope.

## Capabilities

### New Capabilities

- `review-command-compatibility`: Plugin-independent execution of the three
  main review commands across OpenCode V1 and V2, preserving council review
  and mutation safeguards.

### Modified Capabilities

None. Existing OpenSpec capabilities describe provider and release behavior,
not these contributor review commands.

## Impact

- Primary implementation surface: the three command templates under
  `.opencode/commands/`; a small shared Markdown skill may hold repeated
  native delegation and recovery instructions.
- Keep `.opencode/agents/`, `.opencode/uf/packs/`, review policy configuration,
  governance files, embedded plugin implementations, and plugin registrations
  unchanged. Other workflows still consume the plugins; do not remove them
  globally to repair these commands.
- No new runtime packages, V2-only plugin adapter, or OpenCode global
  configuration changes. Existing MCP integrations remain optional where
  current contracts already allow their absence.
- Implementation documentation: `docs/AI_TOOLING.md` and `CHANGELOG.md`;
  assess `README.md` and `AGENTS.md` impact without altering governance gates.
- Add focused compatibility checks and validate actual execution under an
  explicitly recorded V1 version and V2 version. No compatibility claim based
  solely on static template inspection.
- Related upstream work: [UF #714][migration], [UF #797][umbrella], and the
  exact downstream reproduction in [UF #962][downstream]. This change does not
  implement or replace that upstream migration.

[migration]: https://github.com/unbound-force/unbound-force/issues/714
[umbrella]: https://github.com/unbound-force/unbound-force/issues/797
[downstream]: https://github.com/unbound-force/unbound-force/issues/962
