---
name: inferno-test-kit-updater
description: Use when coding agent needs to implement approved Inferno test-kit changes from an impact decision ledger, including tests, fixtures, requirement metadata, obsolete assertions, and source-to-target IG version alignment when creating an updated kit. Trigger for editing Inferno Ruby test-kit files based on test_update_decisions.yaml, implementation reports, or approved IG-change decisions.
---

# Inferno Test Kit Updater

## Purpose

Implement approved decisions in an Inferno test kit. This skill starts only after the impact assessor has produced an implementation-ready decision ledger. When the authorized update creates a target-version kit, align its version references as part of that update.

## Inputs

- Impact decision ledger
- Inferno test kit repository path
- Optional enriched ledger for source evidence and version metadata
- Target decision IDs or all actionable decisions
- For a target-version kit: authoritative source/baseline and target IG versions, package identity, source suite, and target repository/output scope, resolved from user inputs and workflow artifacts

## Workflow

1. Confirm the branch, working tree status, target repository/output location, and decision IDs.
2. Read the relevant decisions and inspect referenced tests, fixtures, helpers, metadata, and generator conventions.
3. Before implementation edits, follow `references/content-guide.md` to resolve source and target identities, discover and classify version references, and define the authorized migration scope. If required versions are missing, contradictory, or ambiguous, stop and ask for clarification. Do not infer the next version or rename versions for an update that does not create a target-version kit.
4. Make the smallest code changes satisfying approved decisions and the authorized version alignment:
   - Add tests only when existing groups cannot naturally cover the change.
   - Revise existing tests when the behavior belongs to an existing runnable.
   - Update fixtures only when required for test behavior.
   - Preserve local Ruby style, Inferno DSL patterns, requirement IDs, and generated conventions; change versioned naming only within the confirmed migration scope.
   - Update affected paths and their references together, following the generated-file policy.
5. Run targeted validation and the version checks below.
6. Produce an implementation report mapping each decision to changes, behavior, validation, and deviations. Record version alignment separately using the additive report extension, including authorization, versions, edits, renames, exclusions, and unresolved cases.

## Guardrails

- Do not implement `manual_review_required` decisions without explicit user approval.
- Creating a target-version kit authorizes necessary version alignment even without a cosmetic version-change decision. Tie those edits to the reported migration scope; do not use that scope to bypass decision approval or implement unrelated behavior.
- Do not otherwise broaden the update beyond the decision ledger without documenting why.
- Never change version identifiers inside `verifies_requirements` or `verify_requirements` declarations, including multiline declarations. Protect complete declarations before edits or regeneration.
- Do not globally replace version strings. Preserve unrelated dependencies, test-kit release versions, baseline references, historical fixtures, and intentional pins; flag ambiguous occurrences for review.
- Follow the generated-file policy in the content guide. Do not remove tests solely because an inventory match was weak; require a decision rationale.

## Validation

- Confirm target-version references are correct throughout the migration scope; scan paths and contents for all discovered source-version forms and classify remaining matches.
- Compare protected declarations against their pre-edit snapshots, including moved and regenerated files; confirm unrelated versions and dependencies were preserved.
- Check renamed paths, requires/imports, modules/classes, identifiers, and registrations for consistency.
- Run syntax checks and relevant test-kit specs/load checks. If checks are unavailable or expensive, run the narrowest meaningful checks and report the gap.
- Tie every changed file to a decision ID, the authorized version migration, or both. Report unresolved occurrences explicitly; do not claim complete alignment while required references remain unresolved.

## References

- Read `references/content-guide.md` before editing, especially version resolution, classification, and protected regions.
- Read `references/implementation-report-contract.md` before summarizing the update.
- Read `references/examples.md` for decision patterns and version-migration examples.
