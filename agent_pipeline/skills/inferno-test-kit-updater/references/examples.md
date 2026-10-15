# Inferno Test Kit Updater Examples

These examples describe implementation patterns. Adapt them to the local test-kit style and the actual decision ledger.

## Revise Allowed Status Assertion

Decision: unauthorized requests may return 401, 403, or 404.

Implementation pattern:

- Locate the existing unauthorized request assertion or helper.
- Replace a single-status expectation with an allowed-status set.
- Add targeted spec cases for 401, 403, 404, and an invalid status.
- Preserve existing failure messages where possible.

Report shape:

```yaml
- change_id: c4bb-001
  decision: revise_existing_test
  status: implemented
  files_changed:
    - lib/example/security_test.rb
    - spec/example/security_test_spec.rb
  behavior_implemented:
    - Unauthorized response assertion now accepts 401, 403, or 404.
    - Added negative coverage for non-allowed status.
  deviations_from_plan: []
  follow_up: []
```

## Add Generated-Pattern Search Test

Decision: add required `service-start-date` EOB search.

Implementation pattern:

- Prefer generator inputs or metadata when the repo regenerates generated tests.
- If manually maintained, add a generated-style `service_start_date_search_test.rb`.
- Wire the test into the EOB group.
- Add or update metadata for the search parameter.
- Add targeted search spec coverage.

Validation:

```yaml
validation:
  commands:
    - command: bundle exec rspec spec/carin_for_blue_button/carin_search_test_spec.rb
      status: passed
  not_run: []
```

## Add Custom Semantic Test

Decision: transportation supportingInfo must be referenced by item.informationSequence.

Implementation pattern:

- Add a focused custom test rather than trying to make a generic generated must-support test understand the semantic rule.
- Use scratch resources already gathered by EOB searches.
- Skip if no relevant resources are available.
- Assert failure only when Transportation supportingInfo exists and no item references its sequence.
- Add positive and negative fixtures/specs.

## Update Fixture Only

Decision: correct APR-DEG typo to APR-DRG.

Implementation pattern:

- Search fixtures, metadata, descriptions, and specs for the old text.
- Update only stale examples or expected strings.
- Do not alter executable behavior unless validation now fails or passes incorrectly.
- Run affected fixture/spec tests.

## Add Attestation

Decision: audit/provenance recording needs coverage but is not directly observable.

Implementation pattern:

- Add an attestation test to the existing visual inspection/security group.
- Link the relevant requirement ID with `verifies_requirements`.
- Use a yes/no input and optional note, matching local attestation style.
- Register the test in the parent group.
- Add a suite load or group registration spec if the kit has one.

## Skip Manual Review

Decision: `manual_review_required` and no explicit user approval.

Implementation report:

```yaml
- change_id: security-privacy-001
  decision: manual_review_required
  status: skipped_manual_review
  files_changed: []
  behavior_implemented: []
  deviations_from_plan:
    - Manual-review decision was not implemented because no explicit approval was provided.
  follow_up:
    - Confirm whether this broad legal/security requirement should become attestation or metadata only.
```

## Align a Target-Version Kit

The user authorizes a target kit for IG `9.0.0`; the linked enriched ledger records baseline `8.0.0` and the intended source suite. Verify those identities against the baseline and target package evidence, then classify occurrences before applying these illustrative mappings:

| Context | Before | After |
| --- | --- | --- |
| Generated directory | `lib/generated/v8.0.0/` | `lib/generated/v9.0.0/` |
| Versioned filename | `us_core_800_test_suite.rb` | `us_core_900_test_suite.rb` |
| Require | `require_relative 'generated/v8.0.0/patient_group'` | `require_relative 'generated/v9.0.0/patient_group'` |
| Local compact module convention | `USCore::V800` | `USCore::V900` |
| Suite registration ID | `us_core_800` | `us_core_900` |
| Underscore identifier | `us_core_8_0_0` | `us_core_9_0_0` |
| Target validator package | `hl7.fhir.us.core#8.0.0` | `hl7.fhir.us.core#9.0.0` |
| Target IG metadata | `ig_version: '8.0.0'` | `ig_version: '9.0.0'` |

These transformations apply only where the reference belongs to the target kit. Keep an independent kit release `VERSION = '0.8.0'`, an unrelated dependency `~> 8.0.0`, and retained baseline suite paths unchanged. Update registrations and consumers with renamed files/modules; check for destination collisions first.

Other confirmed version pairs follow the existing convention without assuming a major increment:

| Source to target | Dotted form | Compact form, if established locally | Underscore form |
| --- | --- | --- | --- |
| `9.0.0` to `9.1.0` | `v9.0.0` to `v9.1.0` | `v900` to `v910` | `us_core_9_0_0` to `us_core_9_1_0` |
| `9.1.0` to `10.0.0` | `v9.1.0` to `v10.0.0` | `v910` to `v1000` | `us_core_9_1_0` to `us_core_10_0_0` |

For another IG, derive its package identity and names from that repository. Do not invent compact encodings for ambiguous multi-digit components or prerelease suffixes. Do not match `8.0.0` inside `18.0.0` or infer version meaning from a substring such as `v8000`.

## Preserve Complete Requirement Calls

Before and after an `8.0.0` to `9.0.0` migration, both declarations below remain byte-for-byte identical, even inside a renamed target-version file:

```ruby
verifies_requirements 'hl7.fhir.us.core_8.0.0@105',
                      'hl7.fhir.us.core_8.0.0@109',
                      'hl7.fhir.us.core_8.0.0@808'

verify_requirements(
  'hl7.fhir.us.core_8.0.0@105',
  'hl7.fhir.us.core_8.0.0@109'
)
```

Snapshot the full calls before editing/regeneration and compare after path translation. Preserve all continuation lines and nested arguments. If regeneration rewrites the calls, restore them safely or report a conflict; do not accept modified historical IDs. A nearby target validator package reference may migrate independently.

## Stop or Exclude Instead of Guessing

- User requests `9.1.0`, but workflow metadata says `10.0.0`: ask which is authoritative before implementation unless the user explicitly superseded the stale metadata.
- No target version is supplied or recoverable: ask; do not infer source major plus one.
- A bug fix stays on the current IG: report version migration as `not_applicable`; do not bump or rename.
- An unexplained `v1110` could encode different version tuples: leave it unchanged, report its location and ambiguity, and ask if it blocks alignment.
- A target path already exists: resolve whether it belongs to the authorized output before moving anything; do not overwrite it.
- An old-version historical fixture remains valid: preserve and document it as an exclusion. Unresolved required target references make the migration partial, even if syntax checks pass.
