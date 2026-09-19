# Generic Skill Prompts

Reusable prompts for FHIR IG comparison, Inferno test-kit planning and updates, and agent-usage reporting. Replace every `[]` placeholder before use.

## Skill 1: fhir-ig-change-ledger
Use the fhir-ig-change-ledger skill.

Compare the local [enter IG package name] package currently named [version number/name] to the local [IG name] currently named [version number/name].

Inputs:
- Old IG zip: [file path]
- New IG zip: [file path]
- Old requirements XLSX for comparison context: [file path if this is available; optional]

Clean test environment:
- Use a fresh artifact directory: pilot/artifacts [enter full file path]
- Use a final products directory: pilot/products [enter full file path]

Run the fhir-ig-change-ledger skill workflow:
1. Extract, convert, and clean the old/new IG narrative from the two US Core zip files.
[if applicable]
2. Compare the cleaned old/new narrative using the XLSX as spreadsheet-assisted context by passing:
   --reqs-xlsx [file path to requirements spreadsheet]
3. Convert the newest generated ig/differences_*.md into a raw IG change ledger

Write/copy the final validated raw ledger to:
- pilot/products/ [enter full file path]

Track uncertainties you have separately and report them in this chat when you are finished. If anything is unclear about these instructions, please request clarification before proceeding.

#### Optional add ons:
- Do not reuse or read existing generated [IG name] artifact directories, including any test kit files, besides the old and new IG zip files listed in this prompt
- Do not use any zip files for any other FHIR IGs


## Skill 2: inferno-test-kit-inventory
Inventory this Inferno test kit repository using the inferno-test-kit-inventory skill.
- Repository: [file path to Inferno test kit repository]
- Baseline suite ID(s): [[suite_id_1[, suite_id_2]]
- Baseline IG/version: [version or “unknown”]
- Output directory: [file path for inventory output]

Requirements:
- Treat the specified suite(s) as the baseline. Do not inventory future/target-version suites unless I explicitly request it.
- Do not modify the test kit repository.
- From the skill directory, ensure its environment is prepared with `uv sync --all-groups`, then run the inventory script.
- Generate the complete deterministic inventory, including:
  - `repos.jsonl`
  - `runnables.jsonl`
  - `requirements.jsonl`
  - `coverage.jsonl`
  - `summary.json`
- Validate that all five files exist.
- Read `summary.json` and report the suite IDs plus counts for runnables, requirements, and coverage rows.
- Confirm the generated inventory contains only the requested baseline suite IDs.
- Briefly inspect representative runnable records and report whether source locations, requirement IDs, profile URLs, FHIR resources, and search parameters were captured where applicable.
- Finish with the output directory path and any limitations or suite-mapping assumptions encountered.

Track uncertainties you have separately and report them in this chat when you are finished. If anything is unclear about these instructions, please request clarification before proceeding.

## Skill 3: inferno-change-evidence-matcher

Use the inferno-change-evidence-matcher skill.
- Input raw change ledger: [change ledger file path]
- Diff file: [file path]

Use the baseline [IG name and version] test kit inventory for evidence searches:
[file path]

And use the baseline test kit requirements spreadsheet for evidence searches:
[file path; if this is available]

Write the enriched ledger to:
[output file path]

Track uncertainties you have separately and report them in this chat when you are finished. If anything is unclear about these instructions, please request clarification before proceeding.

#### Example prompt to update outputs with human review feedback: 
This attached document includes notes from a human review of the accuracy of part of [change ledger name]. The notes are included after "Notes" and "Review Status" under each IG comparison section of the document. Please review the notes and make applicable changes to the file so that it is accurate. Applicable changes may include removing sections that are not relevant for ultimately updating the test kit or correcting test file references. The latter may require you to search the test kit files to identify the accurate relevant test files that should be referenced in the change ledger. If you are not certain of the correct action to take, do not take action for those items and list those uncertainties after completing this revision exercise.

## Skill 4: inferno-test-impact-assessor

Use the inferno-test-impact-assessor skill.

The input enriched change ledger can be found here:
[enriched change ledger with test update decisions included file path]

The JSONL version is available if useful:
[file path for JSONL version]

Use the [IG name and version] baseline test kit source and inventory as the basis for likely files to edit:
[file path to baseline test kit]

Create an implementation-ready decision ledger for the updater skill. Do not edit source files.

Write the decision ledger to:
[decision ledger output file path]

Track uncertainties you have separately and report them in this chat when you are finished. If anything is unclear about these instructions, please request clarification before proceeding.

#### Example prompt to update outputs with human review feedback: 
I have a new version of the input enriched change ledger which is saved at:
[file path] and the JSONL version can be found at: [file path]. Some corrections to existing entries were made and some entries were removed. Please make applicable changes to the decision ledger based on this updated change ledger document. If you are not certain of the correct action to take, do not take action for those items and list those uncertainties in this chat after completing this revision exercise. Do not overwrite the existing ledger, please generate a new version with an updated date stamp.

## Skill 5: inferno-test-kit-updater
You are tasked with identifying the changes in the [IG name and version] baseline test kit that need to be made for a [new version number] test kit. You will not have a [new version number] test kit or any artifacts to use beyond the IG files which have been reviewed using a skill in another conversation. The goal of your exercise is to make the updates necessary to the [baseline version number] test kit that represent the changes found in the new IG version, which are included in the decision ledger. Your target output should be a full [new version number] test kit that aligns with the [new version number] IG.
[decision ledger file path]

Please apply relevant updates based on this decision ledger to this test kit repository:
[test kit file path]

Only implement the changes discussed in the ledger. Keep the edits as small as possible, run targeted validation for the files you touch, and leave an implementation report here:
[output file path]

Do not overwrite the existing candidate test kit files. Create new versions of the files with the updates applied in the repository 
[file path to output directory]

Only use the decision ledger, the candidate repo, and baseline patterns inside the candidate repo to decide what to write.

Track uncertainties you have separately and report them in this chat when you are finished. If anything is unclear about these instructions, please request clarification before proceeding.

## Skill 6: agent-usage-metrics-skill
Create an Agent Usage Metrics report using the agent-usage-metrics skill.

Requested target:
- Target type: <session | conversation | workspace>
- Identifier: <exact rollout filename/session ID, exact conversation title, or workspace name>
- Time scope: <required for workspace: date, date range, latest, or all-time>
- Agent source: <Codex | Claude Code>
- Output directory: <optional output directory>

Requirements:
- Use the skill’s bundled scripts; do not reimplement log parsing or report generation.
- Run the appropriate environment setup and generate the Markdown report with `scripts/run_metrics.py`.
- Resolve the requested target conservatively:
  - An exact rollout filename or session ID means a single `session`.
  - An exact conversation title means the whole `conversation`, including all its sessions.
  - A workspace report must have a date range, `latest`, or explicit `all-time` scope.
- If the request matches multiple possible sessions, conversations, or workspaces, return `needs_clarification`. List only privacy-safe candidate metadata and explain whether I should select a session, conversation, or workspace. Do not silently choose one.
- Include agent-derived activity metrics and attempt ccusage token/cost enrichment unless I explicitly ask to skip it.
- For multi-session conversation or workspace reports, also produce the per-session Markdown breakdown.
- Report the generated Markdown path(s), a concise metrics summary, ccusage precision/status, and any warnings.
