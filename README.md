# promptpack-demo

**A complete AI agent expressed as YAML. No source code.**

This repo is a working demonstration of the [PromptPack](https://promptpack.org) spec and the [PromptKit](https://github.com/AltairaLabs/PromptKit) reference toolkit. The agent — a release-notes composer — is fully declared in YAML. It's tested in CI against real provider models, gated on regressions, and shipped as a content-addressed `.pack.json` artifact on every merge to `main`.

## What's here

```
.
├── config.arena.yaml          # workspace: prompts + providers + scenarios + judges
├── prompts/                   # the agent's behaviour
│   ├── compose.yaml           # raw git log → categorized markdown release notes
│   └── judge-tone.yaml        # LLM-judge that scores release-notes tone
├── providers/                 # which models, what defaults
├── scenarios/                 # tests — input turns + assertions
├── mock-responses.yaml        # fixtures for tokenless local `--providers mock` runs
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

Every PR runs `promptarena` against real provider models (Claude + OpenAI), so the eval exercises real behaviour, and the result is posted back as a comment on the PR. A failed eval marks the PR red. On merge to `main`, `packc compile` builds the pack and publishes it as a **content-addressed GitHub Release** — tagged by the pack's `sha256` — so every shipped version is immutable and traceable. (A mock provider is available for fast, tokenless *local* runs — see above — but it isn't a CI gate, so the PR's pass/fail stays unambiguous.)

The demo PR that makes the value visible:

- **[#1 — Make release notes casual and fun](https://github.com/AltairaLabs/promptpack-demo/pull/1)** (CI red). One-line change to `prompts/compose.yaml` swaps the professional-tone instruction for "be casual and fun, sprinkle in emoji." The `tone-check` scenario carries three independent tone guardrails on the real model output — a `banned_words` validator rejecting emoji glyphs (🎉 🚀 ✨ 🔥), an LLM-judge scoring tone professionalism (must be ≥ `0.7`), and an exclamation-mark regex — and the casual prompt trips them. In a real run the model titled the notes `# Release Notes 🎉`: the `banned_words` validator flagged the emoji and the judge scored it `0.15`, noting the emoji is "casual and unprofessional for an engineering changelog." PR cannot merge until reverted.

> The eval requires repo secrets `ANTHROPIC_API_KEY` and `OPENAI_API_KEY`; without them the job fails fast with credential errors. A mock provider exists for local runs, but it can't catch tone regressions — its output is fixed fixture text that ignores the prompt — which is exactly why the CI gate uses real models. The LLM judge defaults to the mock locally; the CI eval rebinds it to Claude via `override-providers: mock-judge=claude-haiku`.

## The pitch

Today, prompts ship as untested string literals in product code. PromptPack treats agent behaviour the way we treat API services: versioned, tested in CI, gated on PRs, deployed as immutable artifacts.

A prompt change can't reach production without passing eval gates. A model swap doesn't require code changes — just a `providers/` update. A regression is a red check on a PR, not an incident in prod. Same diff-review-test-ship loop you already use for code.

The agent in this repo runs on Claude, OpenAI, or any other provider PromptKit supports. The pack is portable; the runtime executes it.

## Spec version

Pinned to PromptPack [v1.4](https://promptpack.org/docs/spec/versions). For older spec versions, see archived branches.
