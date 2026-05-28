# promptpack-demo

**A complete AI agent expressed as YAML. No source code.**

This repo is a working demonstration of the [PromptPack](https://promptpack.org) spec and the [PromptKit](https://github.com/AltairaLabs/PromptKit) reference toolkit. The agent — a release-notes composer — is fully declared in YAML. It's tested in CI against real provider models, gated on regressions, and shipped as a content-addressed `.pack.json` artifact on every merge to `main`.

## What's here

```
.
├── config.arena.yaml          # workspace: prompts + providers + scenarios + judges
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

Every PR runs `promptarena` twice: once against the mock provider (fast, hermetic fixtures, no tokens) and once against real providers (Claude + OpenAI) so the assertions exercise real model behaviour. Either failure blocks the PR. On merge to `main`, `packc compile` produces a content-addressed `.pack.json` artifact.

The demo PR that makes the value visible:

- **[#1 — Make release notes casual and fun](https://github.com/AltairaLabs/promptpack-demo/pull/1)** (CI red). One-line change to `prompts/compose.yaml` swaps the professional-tone instruction for "be casual and fun, sprinkle in emoji." The `tone-check` scenario fails on three independent assertions against the real model output: an exclamation-mark regex on the response, a `banned_words` validator that rejects emoji glyphs (🎉 🚀 ✨ 🔥), and an LLM-judge backstop scoring tone professionalism below `0.7`. PR cannot merge until reverted.

> Real-provider CI requires repo secrets `ANTHROPIC_API_KEY` and `OPENAI_API_KEY`. Without them, the real-provider job fails fast with credential errors. The mock job still passes — but it can't catch tone regressions, because mock output is hardcoded fixture text that doesn't depend on the prompt. The LLM judge defaults to a mock; the real-provider tier rebinds it to Claude via `override-providers: mock-judge=claude-haiku`.

## The pitch

Today, prompts ship as untested string literals in product code. PromptPack treats agent behaviour the way we treat API services: versioned, tested in CI, gated on PRs, deployed as immutable artifacts.

A prompt change can't reach production without passing eval gates. A model swap doesn't require code changes — just a `providers/` update. A regression is a red check on a PR, not an incident in prod. Same diff-review-test-ship loop you already use for code.

The agent in this repo runs on Claude, OpenAI, or any other provider PromptKit supports. The pack is portable; the runtime executes it.

## Spec version

Pinned to PromptPack [v1.4](https://promptpack.org/docs/spec/versions). For older spec versions, see archived branches.
