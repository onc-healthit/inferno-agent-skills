# Implementation Report Contract

Use this contract after applying a decision ledger to an Inferno test kit.

## Top-Level Shape

```yaml
meta:
  implementation_stage: implemented
  source_decision_ledger: path/to/test_update_decisions.yaml
  test_kit_repo: path/to/inferno-test-kit
  generated_at: YYYYMMDD_HHMMSS
results: []
validation:
  commands: []
  not_run: []
```

## Result Record

```yaml
- change_id: c4bb-001
  decision: revise_existing_test
  status: implemented
  files_changed:
    - lib/example.rb
  behavior_implemented:
    - Added assertion for new required element.
  deviations_from_plan: []
  follow_up: []
```

## Status Values

- `implemented`
- `partially_implemented`
- `not_implemented`
- `skipped_manual_review`

## Additive Version-Migration Record

Existing reports and decision results remain valid unchanged. For new reports, add an optional top-level `version_migration` mapping; include it whenever creating a target-version kit. Do not invent a ledger decision or change existing decision/status values for version alignment.

```yaml
version_migration:
  migration_id: version-alignment-8.0.0-to-9.0.0
  status: implemented
  authorization: User requested a target kit for IG 9.0.0.
  source_ig: {package_id: hl7.fhir.us.core, version: '8.0.0'}
  target_ig: {package_id: hl7.fhir.us.core, version: '9.0.0'}
  version_evidence:
    - location: user request
      establishes: Target package/version and output scope.
    - location: path/to/enriched.yaml#meta.baseline_test_inventory
      establishes: Baseline version and suite; verified against baseline configuration.
  scope:
    source_suite_ids: [us_core_800]
    target_suite_ids: [us_core_900]
    output_root: path/to/target-test-kit
    included_paths: [lib/generated/v9.0.0, lib/us_core_900_test_suite.rb]
  files_changed:
    - lib/us_core_900_test_suite.rb
    - lib/generated/v9.0.0/patient_group.rb
  edits:
    - location: lib/us_core_900_test_suite.rb
      kind: validator_package
      before: hl7.fhir.us.core#8.0.0
      after: hl7.fhir.us.core#9.0.0
      rationale: Package used by the target suite validator.
  renamed_paths:
    - {from: lib/us_core_800_test_suite.rb, to: lib/us_core_900_test_suite.rb, kind: file}
    - {from: lib/generated/v8.0.0, to: lib/generated/v9.0.0, kind: directory}
  protected_references:
    - source_location: lib/generated/v8.0.0/patient_group.rb#verifies_requirements
      target_location: lib/generated/v9.0.0/patient_group.rb#verifies_requirements
      declaration_count: 1
      comparison: unchanged
      verification: Complete declaration compared with pre-edit snapshot.
  exclusions:
    - location: lib/us_core_test_kit/version.rb
      classification: independent_release_version
      reason: Kit release version is independent of the target IG.
  unresolved: []
```

Use the actual confirmed versions and conventions; the example does not prescribe US Core or a version pair. Record enough evidence to distinguish IG/package identities from kit releases, dependencies, and historical requirement sets. List content/identifier/config edits and renames with before/after values and rationale, grouping repetitive changes only when all affected paths remain traceable. Record copy-to-target paths as copies rather than renames when the baseline is retained.

For `version_migration.status`, use `implemented`, `partially_implemented`, or `not_implemented`; `not_applicable` is also allowed for operations that do not create a target-version kit. For a not-applicable record, retain only the migration status and explanation as needed; do not invent version values. Missing migration data in an older report means unspecified, not validated alignment.

Record protected declarations of both names, including multiline calls, their comparison outcome, and snapshot evidence (or exact before/after declarations). Account for every original declaration across moved/regenerated files. Record zero declarations checked explicitly when none exist. Put every remaining source-version occurrence in exclusions with a reason, or in `unresolved` with location, reference, issue, and needed resolution. A changed protected declaration or unresolved required version reference prevents `implemented` migration status.

Use the existing top-level `validation.commands` and `validation.not_run` for version scans, protected-region comparisons, dependency/exclusion review, path/registration checks, syntax, and tests. Record commands/outcomes and any manual checks with their inspected scope and result. Do not imply that a passing syntax check proves complete version alignment.

## Rules

- Tie every changed file to a decision ID, the authorized `version_migration.migration_id`, or both.
- Record validation commands exactly as run.
- If validation cannot run, explain why and identify residual risk.
- Keep report factual; do not restate the entire ledger.
