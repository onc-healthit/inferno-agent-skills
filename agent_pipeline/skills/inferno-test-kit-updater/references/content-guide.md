# Inferno Test Kit Updater Content Guide

Implement approved decision-ledger records and, when creating a target-version kit, its authorized version alignment. Leave a factual implementation report.

## Implementation Principles

- Implement only approved behavior decisions; do not implement `manual_review_required` without explicit approval.
- Tie changed files to decision IDs, the authorized version migration, or both.
- Prefer existing local patterns and the smallest change satisfying expected behavior.
- Preserve Inferno DSL style, requirement-link conventions, and naming patterns. Change versioned modules/classes and runnable IDs only when classified as part of the target kit, with all callers and registrations updated.

## Resolve Versions and Scope Before Editing

1. Establish whether the request creates a target-version kit or changes an existing kit without a version migration. For the latter, do not invent an increment or rename; report migration as not applicable.
2. Check explicit user inputs and relevant upstream artifacts. Follow the decision ledger's `meta.source_enriched_ledger` when available. Version evidence can include enriched `meta.baseline_test_inventory.ig_version`, `meta.baseline_test_inventory.suite_ids`, `meta.target_ig_version`, and raw-ledger `meta.old_ig` / `meta.new_ig` package identities and versions. These are optional evidence fields, not new upstream requirements.
3. Verify the source package/version against the intended baseline suite, repository metadata, validator/package configuration, and generator inputs. Verify the target package/version against the intended target IG artifacts or explicit user instruction; a target implementation need not exist. Record evidence locations and the repository/output scope. User clarification can explicitly supersede stale artifact metadata; otherwise missing, conflicting, or ambiguous identities require clarification before edits.
4. Keep IG/package versions, test-kit release versions, dependency versions, and historical requirement-set IDs separate. Never infer a target by adding one to the source major version. Do not bump a kit release or dependency merely because the IG version changes.
5. Define included suites, shared files, generator inputs, and output paths. Preserve baseline suites that must coexist with the target. Use ledger target-output hints as discovery guidance, not authoritative version inputs. Trace baseline decision paths to target paths without rewriting the evidence ledger.

## Discover and Classify Version References

Inspect the candidate target tree, generator inputs, and loading/configuration files that reference it. Search both path names and file contents, including relevant configuration files normally hidden or ignored. Do not traverse Git internals, dependencies, or unrelated build artifacts.

Inventory dotted and prefixed versions, compact/underscore forms, and versioned names observed locally. Include directory names, filenames, Ruby modules/classes/constants, runnable identifiers, requires/imports, registrations, package references, validator settings, metadata, and documentation describing the target kit.

Classify every occurrence by its meaning and record its location, representation, action, and rationale:

| Classification | Action |
| --- | --- |
| Target kit or IG identity/configuration | Migrate within the authorized scope; update its consumers together. |
| Protected requirement declaration | Preserve the complete declaration exactly; see below. |
| Historical/baseline reference or fixture | Preserve unless a separate approved behavior decision requires a change; protected declarations remain immutable. |
| Independent release/dependency version or intentional pin | Preserve unless explicitly authorized for that distinct purpose. A source IG package pin driving target validation is target configuration, not an unrelated pin. |
| Ambiguous ownership or representation | Leave unchanged and flag for review; ask before dependent edits if it prevents a reliable migration. |

Numeric equality alone is not evidence of ownership. A baseline folder or old-version compatibility test may correctly remain in a multi-version repository.

## Apply Version Transformations

- Derive source/target representation pairs from the confirmed full versions and local conventions: dotted, `v`-prefixed, underscore-separated, compact numeric, and Ruby constant/module forms. Examples are in `examples.md`; none is a universal naming rule.
- Use exact contextual matches with boundaries appropriate to paths, Ruby identifiers, or configuration values. Do not replace `8.0.0` inside `18.0.0` or replace digits throughout arbitrary text. Never use an unconstrained repository-wide replacement.
- Compact forms may be ambiguous for multi-digit components or prerelease/build versions. Do not strip components or suffixes or invent encodings. Use an established unambiguous convention or ask for clarification.
- Plan old-to-new file and directory paths before renaming. Check destination collisions and existing target suites; do not overwrite or merge them implicitly. Limit renames to the chosen output scope.
- Update requires/imports, references, suite/group/test IDs, modules/classes/constants, configuration, package metadata, and registrations that consume migrated names. Keep unrelated identifiers stable. If a consumer is inside a protected declaration, leave it unchanged and report the conflict.
- Apply version alignment under the user's target-kit authorization even if absent from cosmetic ledger entries. Record it separately from behavior decisions and preserve their approval requirements.

## Protected Requirement Declarations

Treat `verifies_requirements` and `verify_requirements` declarations as protected regions. Never migrate any version identifier inside them, including quoted strings, symbols, arrays, interpolations, or nested expressions.

- Before any edit or regeneration, snapshot complete existing declarations in affected files, recording file and declaration identity so they can be compared after moves.
- Identify the full Ruby call, with or without parentheses and across multiline argument lists. Include continuations, nested delimiters, comments, and heredoc arguments where present. A line-only regex is insufficient; use Ruby-aware inspection or parsing, or careful manual inspection. If the boundary is uncertain, stop the transformation of that region and flag it.
- Preserve each existing declaration byte-for-byte. Do not replace just the first line or temporarily mask only the version substring. New requirement links needed by an approved behavior decision must use authoritative IDs, never IDs synthesized by version substitution.
- Compare snapshots after edits and regeneration, mapping old to new paths and accounting for every existing declaration. If regeneration changes a protected declaration, restore it only when doing so is consistent with the intended kit; otherwise stop and report the conflict. Do not silently accept its modification or deletion.

## Generated File Policy

- If the repo normally regenerates from IG packages, prefer updating the scoped generator/package inputs and regenerating using the project's process. Snapshot protected declarations first and review regenerated output for unrelated changes.
- If regeneration cannot run, document the needed step and validation gap; do not claim the target output is fully aligned while required generated changes remain outstanding.
- If generated output is source-controlled and manually maintained, edit narrowly and preserve generated style.
- Keep custom semantic behavior in existing helpers/custom groups where appropriate rather than duplicating it across generated files. Keep generated and custom responsibilities separate.

## Common Edit Locations

- Generated search/read/validation/must-support files and their generator inputs.
- Shared helpers, custom semantic/SMART/security/workflow groups, and attestation groups.
- Fixtures and specs needed for changed behavior.
- Requirement metadata/docs for traceability-only decisions, subject to protected declarations.
- Target-version directories, filenames, Ruby identifiers, package/validator configuration, and loading/registration files.

## Validation Strategy

1. Review the diff against the approved behavior decisions and version scope, including old-to-new paths.
2. Confirm target identities and configuration match the resolved target version. Scan paths and contents for every discovered source representation; each remaining match must have a classification and preservation rationale or an unresolved entry. Check target forms for incorrect/partial replacements too.
3. Compare all protected snapshots exactly, including moved/regenerated files. Review excluded references and unrelated release/dependency versions for preservation.
4. Check renamed paths and requires/imports, module/class references, runnable IDs, suite/group registrations, and package metadata together. Run Ruby syntax checks, targeted specs, and suite load/registration checks as supported; validate fixtures when changed.
5. Use broader tests when targeted checks are insufficient. Record exact commands, outcomes, environment failures, checks not run, and residual risk.

Unresolved required version references or changed protected declarations prevent reporting a complete migration. Record ambiguous occurrences explicitly even when approved behavior edits can proceed independently.

## Report Expectations

Use `implementation-report-contract.md` for decision results and the additive version-migration record. Include authorization and version evidence, scoped edits/renames, protected and other excluded references, validation, and unresolved cases. Do not restate the whole decision ledger.
