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
2. Unpack and inspect both packages directly. Locate narrative-bearing IG artifacts, including rendered pages, markdown or HTML source, StructureDefinitions, CapabilityStatements, examples, value sets, and requirement or conformance tables when present. Ignore generated navigation, timestamps, and other presentation-only noise.
3. Normalize only enough to compare meaning: preserve headings, anchors, requirement IDs, resource names, element paths, cardinalities, bindings, MUST/SHALL language, and links to authoritative artifacts. Do not silently discard substantive prose because it is hard to map.
4. Match old and new material by stable identifiers first, then canonical URLs, anchors, logical resource/element context, and finally clearly labeled narrative similarity. Identify additions, removals, modifications, moves, splits, merges, and uncertain matches. Use the optional XLSX only to add historical context and mark its contribution.
5. Only include requirement-level changes in the ledger that would ultimately impact the existing test kit to the best of your knowledge. Do not infer a change from cosmetic, navigational, or other unimportant differences. If a requirement is unchanged but its narrative context has changed, record the context change in the diff but do not create a ledger entry.
6. Write the narrative diff before creating the ledger. Each ledger record must trace to one or more diff entries; do not infer a change absent from the diff.

Do the comparison yourself using the local contents.

## Narrative-diff artifact contract

Write `<output_dir>/differences_<old_version>_to_<new_version>.md`. The filename may use sanitized versions or a timestamp when either version is unavailable.

The document must contain:

- A metadata section with the two IG identities, input locations, comparison date, optional XLSX location, and disclosed limitations.
- A summary table counting added, removed, modified, moved, split, merged, and unresolved entries.
- One uniquely identified entry per requirement-level narrative change, organized by IG artifact and requirement context. Include the change type; old and new source locations; requirement ID(s), if available; resource and element context; verbatim or tightly bounded old/new text; a concise factual summary; and confidence (`high`, `medium`, or `low`).
- An explicit `Unresolved or non-comparable material` section for missing, ambiguous, generated-only, or unreadable source material. State why it was not compared and what evidence would resolve it.

Entries must be independently reviewable: a reviewer must be able to find the cited source content without rerunning the comparison. Do not characterize cosmetic rendering, navigation, date, or build-output differences as requirement changes.

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

Every `changes` item must include `change_id`, `diff_entry_id`, `artifact_id`, `artifact_type`, `source_locations`, `requirement_ids`, `affected_resource`, `element_paths`, `change_type`, `old_text`, `new_text`, `summary`, `confidence`, and `source_of_truth_status`. Use `null` or an empty list where evidence is absent; never invent values. `change_id` must be stable for the same sources and diff entry.

Every `unresolved` item must include `unresolved_id`, `source_locations`, `reason`, `available_evidence`, and `needed_to_resolve`. The ledger may additionally carry IG-native conformance facts such as old/new cardinality, binding, actor, or scope when directly evidenced by the diff.

The ledger is IG-only. It must not contain `inventory_match`, `candidate_tests`, `candidate_coverage`, test or suite names, implementation actions, prioritization, decisions, or implementation notes.

## Validation and handoff

Before reporting completion:

- Confirm both inputs were inspected and their identities and locations are recorded, including any unavailable metadata.
- Confirm the Markdown diff exists, is non-empty, has the required metadata, summary, uniquely identified entries, and unresolved section, and that every entry cites old/new source locations or explains why one side is absent.
- Confirm the YAML parses; `meta.ledger_stage` is exactly `raw_ig_change_ledger`; declared totals equal the lengths of `changes` and `unresolved`; every `diff_entry_id` resolves to a Markdown diff entry; and all required record fields are present.
- Confirm the YAML contains only IG evidence and no downstream Inferno matching or implementation decisions.
- Report the paths of both artifacts, counts by change type, unresolved count, and material limitations. Do not proceed to downstream matching or implementation work.
