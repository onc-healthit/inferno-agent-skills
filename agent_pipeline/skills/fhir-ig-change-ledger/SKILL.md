---
name: fhir-ig-change-ledger
description: Compare two FHIR Implementation Guide packages directly and produce a human-reviewable requirement-level narrative diff plus an IG-only raw YAML change ledger. Use for old/new IG ZIPs or package URLs, optionally with an old requirements XLSX.
---

# FHIR IG Change Ledger

## Purpose and boundary

Create two reproducible, reviewable artifacts from an old/new FHIR IG comparison:

1. A requirement-level narrative diff for human review.
2. A raw YAML change ledger derived only from that diff.

This skill ends at IG evidence. Do not search for, match, name, or assess Inferno tests, suites, coverage, or source code. Do not recommend, prioritize, or make implementation decisions. Do not delegate the comparison or ledger generation to pre-existing automation.

## Inputs

Accept all of the following:

- An old IG ZIP file or package URL.
- A new IG ZIP file or package URL.
- An output directory.
- Optionally, an old requirements XLSX that provides historical requirement identifiers or wording.

Resolve package URLs locally, retain the resolved source URL and package identity in the artifacts, and inspect the package contents directly. Treat the XLSX as contextual evidence only: it may help identify a renamed, retired, or split requirement, but it does not replace the old IG as the source of truth.

## Direct comparison workflow

1. Establish identity for each IG: package ID, version, canonical URL when present, and source path or URL. Record unavailable values explicitly.
2. Unpack both packages. Find requirement pages and FHIR resources, including profiles, CapabilityStatements, terminology, extensions, search parameters, operations, and conformance tables.
3. Make a file manifest before comparing details. Add one row for every candidate file from either version: old file, new file, type, match reason, status, and notes. Use `matched`, `added`, `removed`, `moved`, `split`, `merged`, `excluded`, or `unresolved`. Every candidate file needs a row.
4. Ignore presentation noise unless it contains a unique requirement. This includes navigation, tables of contents, indexes, history and download pages, QA/build pages, repeated headers and footers, timestamps, and duplicate JSON/XML/TTL views. Treat translated or duplicate rendered pages as one file unless their requirement text differs. Skip examples, mappings, and guidance only when they add no normative requirement. Record why each file was skipped. If unsure, mark it `unresolved`.
5. Clean formatting, not meaning. Remove repeated navigation, headers, footers, timestamps, and whitespace-only changes. Keep exact requirement wording, IDs, URLs, headings, anchors, resource and element paths, cardinalities, bindings, must-support flags, actors, scope, conditionality, and normative references. When prose and a FHIR resource state the same rule, use the FHIR resource as the technical source and cite the prose as supporting context. If a constraint disappears from a profile differential, compare the base definition or snapshot before calling it removed or changed. If the effective constraint is still unclear, mark it `unresolved`.
6. Match files in this order:
   1. IDs and canonical URLs.
   2. Resource type, artifact name, or element path.
   3. Headings and nearby requirement text.
   4. Similar wording.

   Label the last kind as an inferred match and record the evidence. Before calling a file added or removed, search both versions for a renamed, moved, split, or merged equivalent.
7. Compare independent requirement sections separately. Record added, removed, changed, moved, split, merged, and uncertain requirements. Include every substantive IG change. Do not decide whether it affects a test kit.
8. If an old requirements XLSX is supplied, record the sheets and columns used. Preserve IDs and, when available, requirement text, URL, conformance, actor, scope, conditionality, and planning metadata. Use it as historical context only. If it conflicts with the IG, follow the IG and record the limitation.
9. Write the narrative diff before creating the ledger. Each ledger record must trace to one or more diff entries; do not infer a change absent from the diff.

Do the comparison yourself using the local contents.

## Narrative-diff artifact contract

Write `<output_dir>/differences_<old_version>_to_<new_version>.md`. The filename may use sanitized versions or a timestamp when either version is unavailable.

The document must contain:

- A metadata section with the two IG identities, input locations, comparison date, optional XLSX location, and disclosed limitations.
- The artifact manifest, including match basis and a disposition for every candidate artifact from both versions.
- A summary table counting added, removed, modified, moved, split, merged, and unresolved entries.
- One uniquely identified entry per requirement-level narrative change, organized by IG artifact and requirement context. Include the change type; old and new source locations; requirement ID(s), if available; resource and element context; verbatim or tightly bounded old/new text; and a concise factual summary.
- An explicit `Unresolved or non-comparable material` section for missing, ambiguous, generated-only, or unreadable source material. State why it was not compared and what evidence would resolve it.

Entries must be independently reviewable: a reviewer must be able to find the cited source content without rerunning the comparison. Use `package-path#anchor` for narrative pages and `package-path#/JSON-pointer` for structured FHIR artifacts. Do not characterize cosmetic rendering, navigation, date, or build-output differences as requirement changes.

## Raw-ledger artifact contract

Write `<output_dir>/change_ledger_raw_<old_version>_to_<new_version>.yaml`. It must be valid YAML with this top-level shape:

```yaml
meta:
  ledger_stage: raw_ig_change_ledger
  old_ig: { package_id: null, version: null, canonical: null, source: null }
  new_ig: { package_id: null, version: null, canonical: null, source: null }
  narrative_diff: differences_old_to_new.md
  optional_old_requirements_xlsx: null
  total_changes: 0
  total_unresolved: 0
changes: []
unresolved: []
```

Every `changes` item must include `change_id`, `diff_entry_id`, `artifact_id`, `artifact_type`, `source_locations`, `requirement_ids`, `affected_resource`, `element_paths`, `change_type`, `old_text`, `new_text`, `summary`, and `source_of_truth_status`. Use `null` or an empty list where evidence is absent; never invent values. `change_id` must be stable for the same sources and diff entry.

Use one of these `change_type` values: `requirement_added`, `requirement_removed`, `requirement_modified`, `conformance_changed`, `cardinality_changed`, `binding_or_terminology_changed`, `actor_or_scope_changed`, `search_or_operation_changed`, `artifact_moved_or_renamed`, `requirement_split_or_merged`, or `other_substantive_change`. Use `other_substantive_change` only when no preferred value fits and explain why in `summary`. Construct a stable `change_id` from the package identities, artifact ID or canonical URL, source locations, and change type; do not use run order alone.

Every `unresolved` item must include `unresolved_id`, `source_locations`, `reason`, `available_evidence`, and `needed_to_resolve`. The ledger may additionally carry IG-native conformance facts such as old/new cardinality, binding, actor, or scope when directly evidenced by the diff.

The ledger is IG-only. It must not contain `inventory_match`, `candidate_tests`, `candidate_coverage`, test or suite names, implementation actions, prioritization, decisions, or implementation notes.

## Validation and handoff

Before reporting completion:

- Confirm both inputs were inspected and their identities and locations are recorded, including any unavailable metadata.
- Confirm every candidate artifact from both versions is `matched`, `excluded` with a reason, or `unresolved`, and report counts for each manifest status.
- Confirm the Markdown diff exists, is non-empty, has the required metadata, summary, uniquely identified entries, and unresolved section, and that every entry cites old/new source locations or explains why one side is absent.
- Confirm the YAML parses; `meta.ledger_stage` is exactly `raw_ig_change_ledger`; declared totals equal the lengths of `changes` and `unresolved`; every `diff_entry_id` resolves to a Markdown diff entry; and all required record fields are present.
- Confirm every `change_type` uses the preferred vocabulary and every `change_id` follows the stable-ID rule.
- Confirm the YAML contains only IG evidence and no downstream Inferno matching or implementation decisions.
- Report the paths of both artifacts, counts by change type, unresolved count, and material limitations. Do not proceed to downstream matching or implementation work.
