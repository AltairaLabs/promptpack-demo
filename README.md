# promptpack-demo

**A complete AI agent expressed as YAML. No source code.**

This repo is a working demonstration of the [PromptPack](https://promptpack.org) spec and the [PromptKit](https://github.com/AltairaLabs/PromptKit) reference toolkit. The agent — a release-notes composer — is fully declared in YAML. It's tested in CI against real provider models, gated on regressions, and shipped as a content-addressed `.pack.json` artifact on every merge to `main`.

## What's here

```
.
├── config.arena.yaml          # workspace: prompts + providers + scenarios + workflow
├── prompts/                   # the agent's behaviour
│   ├── extract.yaml           # commit → structured change
│   ├── categorize.yaml        # change → feature | fix | breaking
│   └── compose.yaml           # categorized → markdown release notes
├── providers/                 # which models, what defaults
├── scenarios/                 # tests — input turns + assertions
├── mock-responses.yaml        # hermetic CI fixtures (no API tokens needed)
└── .github/workflows/pack-ci.yml
```

That's the whole agent. No Python, no TypeScript, no framework glue.

## Run it locally

```bash
npm install -g @altairalabs/promptarena @altairalabs/packc

# Inspect the resolved workspace
promptarena config-inspect

# Run the scenario against the mock provider (hermetic, no tokens needed)
promptarena run --scenario scenarios/tone-check.scenario.yaml --providers mock

# Run against real providers (requires .env with API keys)
cp .env.example .env  # fill in ANTHROPIC_API_KEY, OPENAI_API_KEY
promptarena run --scenario scenarios/tone-check.scenario.yaml --providers claude-haiku,openai-mini

# Compile to a deployable artifact
packc compile -c config.arena.yaml -o dist/release-notes.pack.json
```

## What the CI proves

Every PR runs `promptarena` against the mock provider — fast, deterministic, no tokens. Failures block the PR. On merge to `main`, `packc compile` produces a content-addressed `.pack.json`, and that artifact is attached to a GitHub Release tagged by version.

Two demo PRs in this repo's history make the value visible:

- **#1 — Initial agent** (CI green). The baseline pack passes all scenarios.
- **#2 — Make the tone casual** (CI red). One-line change to `prompts/compose.yaml` adds "be casual and fun, sprinkle in emoji." The `tone-check` scenario fails on three independent assertions: a regex for emoji characters, a banned-words validator on the prompt, and an LLM-judge backstop scoring tone professionalism. PR cannot merge until reverted.

## The pitch

Today, prompts ship as untested string literals in product code. PromptPack treats agent behaviour the way we treat API services: versioned, tested in CI, gated on PRs, deployed as immutable artifacts.

A prompt change can't reach production without passing eval gates. A model swap doesn't require code changes — just a `providers/` update. A regression is a red check on a PR, not an incident in prod. Same diff-review-test-ship loop you already use for code.

The agent in this repo runs on Claude, OpenAI, or any other provider PromptKit supports. The pack is portable; the runtime executes it.

## Spec version

Pinned to PromptPack [v1.4](https://promptpack.org/docs/spec/versions). For older spec versions, see archived branches.
