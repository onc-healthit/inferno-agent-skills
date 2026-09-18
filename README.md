# Inferno Agent Skills

This project extends upon the initial [ONCLAIVE repository](https://github.com/onc-healthit/ONCLAIVE).

The purpose of this project is to aid Inferno test-kit developers keep conformance tests aligned with updated FHIR Implementation Guides (IGs). The skills turn an old/new IG comparisons into a structured change ledger, connect each change to evidence in a baseline Inferno test kit, and produce a reviewable decision about whether a test, fixture, requirement link, or assertion needs to change. Approved decisions can then guide a focused implementation and validation pass instead of requiring developers to rediscover the relevant IG and test-kit context by hand.

It is designed for a human-in-the-loop workflow with an agent client such as Codex or Claude Code. The user provides the source materials, target test kit, scope, credentials, and approvals; the agent gathers evidence and prepares artifacts at each stage; and the developer reviews the findings and remains responsible for implementation decisions. The usage-metrics skill and dashboard provide a separate way to review the effort and agent activity associated with this work.

This repository uses one [uv](https://docs.astral.sh/uv/) environment for the FHIR IG-change pipeline, the agent-usage-metrics skill, and the Streamlit evaluation dashboard.

## Initial setup

Install uv if it is not already available, then run from the repository root:

```bash
uv python install 3.12
uv sync --all-groups
```

## Install the skills in Codex

From this repository's root, install the dependencies and link the skills into Codex:

```bash
uv sync --all-groups
mkdir -p "$HOME/.agents/skills"
ln -s "$(pwd)"/agent_pipeline/skills/* "$HOME/.agents/skills/"
```

Restart Codex, then invoke a skill by name, such as `$fhir-ig-change-ledger`.

## Skill flow

For IG-driven Inferno test-kit updates, the recommended skill workflow is:

1. `fhir-ig-change-ledger` — compare IG versions and create a raw change ledger.
2. `inferno-test-kit-inventory` — create a baseline inventory of the target test kit.
3. `inferno-change-evidence-matcher` — match ledger changes to inventory and source evidence.
4. `inferno-test-impact-assessor` — turn matched changes into an implementation decision ledger.
5. `inferno-test-kit-updater` — apply approved decisions and validate the test-kit changes.

For agent-use evaluation, run `agent-usage-metrics-skill` to generate reports, then open the Agent Evaluation Dashboard to visualize them.

The IG-change-ledger skill calls an LLM and requires a provider key: `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, or, for comparison, `GEMINI_API_KEY`. Set keys in your shell or an untracked `.env` file. The other skills run locally, although agent-usage token and cost enrichment may require separately approving a `ccusage` download.
