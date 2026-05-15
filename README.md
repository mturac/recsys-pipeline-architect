# recsys-pipeline-architect

> A Claude skill for designing composable recommendation, ranking, and feed pipelines — built around the six-stage **Source → Hydrator → Filter → Scorer → Selector → SideEffect** framework popularized by xAI's open-sourced [For You algorithm](https://github.com/xai-org/x-algorithm).

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Skill: Claude Code](https://img.shields.io/badge/skill-Claude%20Code-orange.svg)](#installing-the-skill)
[![Examples](https://img.shields.io/badge/examples-TypeScript%20%7C%20Go%20%7C%20Python-2ea44f.svg)](#examples)
[![Pattern: Apache 2.0](https://img.shields.io/badge/pattern%20source-Apache%202.0-lightgrey.svg)](https://github.com/xai-org/x-algorithm)

<p align="center">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 750 200" width="100%" role="img" aria-label="Six-stage recsys pipeline: Source, Hydrator, Filter, Scorer, Selector, SideEffect">
  <text x="375" y="24" text-anchor="middle" style="font:600 14px -apple-system,'Segoe UI',Helvetica,sans-serif;fill:#111827">Six-stage recsys pipeline</text>
  <text x="375" y="42" text-anchor="middle" style="font:400 11px -apple-system,'Segoe UI',Helvetica,sans-serif;fill:#6b7280">top-K items for any (user, context)</text>

  <rect x="20"  y="70" width="110" height="70" rx="6" style="fill:#eff6ff;stroke:#1f2937;stroke-width:1.5"/>
  <rect x="140" y="70" width="110" height="70" rx="6" style="fill:#eff6ff;stroke:#1f2937;stroke-width:1.5"/>
  <rect x="260" y="70" width="110" height="70" rx="6" style="fill:#fefce8;stroke:#1f2937;stroke-width:1.5"/>
  <rect x="380" y="70" width="110" height="70" rx="6" style="fill:#fefce8;stroke:#1f2937;stroke-width:1.5"/>
  <rect x="500" y="70" width="110" height="70" rx="6" style="fill:#ecfdf5;stroke:#1f2937;stroke-width:1.5"/>
  <rect x="620" y="70" width="110" height="70" rx="6" style="fill:#faf5ff;stroke:#1f2937;stroke-width:1.5"/>

  <g style="font:700 11px -apple-system,'Segoe UI',Helvetica,sans-serif;fill:#9ca3af">
    <text x="30"  y="88">1</text>
    <text x="150" y="88">2</text>
    <text x="270" y="88">3</text>
    <text x="390" y="88">4</text>
    <text x="510" y="88">5</text>
    <text x="630" y="88">6</text>
  </g>

  <g style="font:600 13px -apple-system,'Segoe UI',Helvetica,sans-serif;fill:#111827">
    <text x="75"  y="106" text-anchor="middle">Source</text>
    <text x="195" y="106" text-anchor="middle">Hydrator</text>
    <text x="315" y="106" text-anchor="middle">Filter</text>
    <text x="435" y="106" text-anchor="middle">Scorer</text>
    <text x="555" y="106" text-anchor="middle">Selector</text>
    <text x="675" y="106" text-anchor="middle">SideEffect</text>
  </g>

  <g style="font:500 10px -apple-system,'Segoe UI',Helvetica,sans-serif;fill:#6b7280">
    <text x="75"  y="124" text-anchor="middle">parallel</text>
    <text x="195" y="124" text-anchor="middle">parallel</text>
    <text x="315" y="124" text-anchor="middle">sequential</text>
    <text x="435" y="124" text-anchor="middle">sequential</text>
    <text x="555" y="124" text-anchor="middle">top-K</text>
    <text x="675" y="124" text-anchor="middle">async</text>
  </g>

  <g style="fill:#9ca3af">
    <polygon points="132,102 138,105 132,108"/>
    <polygon points="252,102 258,105 252,108"/>
    <polygon points="372,102 378,105 372,108"/>
    <polygon points="492,102 498,105 492,108"/>
    <polygon points="612,102 618,105 612,108"/>
  </g>

  <text x="375" y="172" text-anchor="middle" style="font:400 11px -apple-system,'Segoe UI',Helvetica,sans-serif;fill:#6b7280">candidates flow left → right · side effects fire async, never blocking the response</text>
</svg>
</p>

## Why this exists

Most "recommendation systems" in production aren't exotic ML — they're **pipelines**. You fetch candidates from one or more sources, enrich them with metadata, drop the ineligible, score the rest, sort, pick the top K, then fire off async side effects. The scoring model and the items change. The pipeline shape doesn't.

xAI open-sourced this exact shape in 2024 with their For You algorithm ([Apache 2.0](https://github.com/xai-org/x-algorithm)). This skill turns it into a reusable recipe and applies it well beyond social feeds — Strapi content CMSs, RAG rerankers, task prioritizers, notification triage, search reranking, ad selection: anywhere you need *"the top K items for this (user, context)."*

When you invoke the skill, Claude walks you through eight steps (use case → sources → hydrations → filters → scorers → selector → side effects → scaffold), surfaces the architectural trade-offs you'd otherwise default through silently (multi-action vs single-score, candidate isolation vs joint scoring, online vs offline), and emits a runnable scaffold in your stack.

## What's in the box

- **`SKILL.md`** — the skill itself. Drop into `~/.claude/skills/recsys-pipeline-architect/` for Claude Code, or paste into a Claude.ai project's custom instructions.
- **`references/`** — load-on-demand deep dives: interface definitions in 4 languages, the multi-action scoring pattern, candidate isolation, a filter cookbook (12 patterns), a scorer cookbook (weighted sum, MMR, diversity penalty, position debiasing).
- **`examples/`** — three runnable scaffolds in different stacks, every one green on its test suite.

## The six-stage pattern

| # | Stage | Job | Parallel? |
|---|---|---|---|
| 1 | Source | Fetch candidates from one or more origins | Yes |
| 2 | Hydrator | Enrich candidates with metadata | Yes |
| 3 | Filter | Drop ineligible candidates | No (sequential) |
| 4 | Scorer | Assign scores | No (chain order matters) |
| 5 | Selector | Sort and pick top K | Single op |
| 6 | SideEffect | Cache, log, emit events, update served-history | Async (non-blocking) |

The skill walks you through each stage, surfaces the trade-offs (multi-action vs single-score, candidate isolation vs joint scoring, online vs offline batch), and generates a runnable scaffold in your stack.

## Examples

### Strapi v5 content feed — TypeScript

`examples/strapi-content-feed/` — a Strapi plugin that adds `GET /api/feed/for-you` to any Strapi instance with an `article` content type. Multi-action scoring with `P(read)`, `P(like)`, `P(share)`, `P(skip)`. Author diversity. Standard filters. Jest tests cover stage ordering, parallel source fan-out, and source-error tolerance.

### Zentra-compatible pipeline — Go

`examples/zentra-go/` — Go implementation packaged as a Zentra `engine.Module`. The `pipeline/` package is standalone and usable outside Zentra. Uses generics for type-safe candidate flows; tests exercise filtering, sorting, parallel sources, and error survival.

### PMAI task prioritizer — Python / FastAPI

`examples/pmai-task-prioritizer/` — applies the pattern to task ranking. `GET /tasks/next?user_id=42&limit=10` returns the top tasks for a user based on priority, due date, in-progress status, and project diversity. Verified runnable: includes a pytest suite and a working FastAPI endpoint.

## Installing the skill

### Claude Code

```bash
mkdir -p ~/.claude/skills/
git clone https://github.com/mturac/recsys-pipeline-architect.git \
  ~/.claude/skills/recsys-pipeline-architect
```

Then in a Claude Code session:

```
/skill recsys-pipeline-architect
```

Or just describe a recsys/ranking/feed problem and Claude Code will load the skill via its trigger keywords.

### Claude.ai (chat)

1. Open a Claude project.
2. Paste the contents of `SKILL.md` into the project's custom instructions.
3. Upload the `references/` files as project knowledge so they load on demand.

## Trying the examples

### Strapi

```bash
cd examples/strapi-content-feed
npm install
npm test
```

Integrating into a real Strapi v5 project: see `examples/strapi-content-feed/README.md`.

### Go

```bash
cd examples/zentra-go
go test ./pipeline/
go run examples/ranking_demo.go
```

### Python

```bash
cd examples/pmai-task-prioritizer
pip install -e .
pip install pytest pytest-asyncio
pytest tests/
uvicorn api:app --reload
curl 'http://localhost:8000/tasks/next?user_id=42&limit=5'
```

## Attribution

The six-stage pipeline pattern (Source → Hydrator → Filter → Scorer → Selector → SideEffect), the multi-action scoring approach, and the candidate isolation rule are inspired by xAI's open-sourced X For You algorithm:

https://github.com/xai-org/x-algorithm (Apache 2.0)

This repository is an independent reimplementation of the *pattern* in TypeScript, Go, and Python. No code is copied from the original repo. The skill and examples are licensed MIT.

## License

MIT — see [LICENSE](./LICENSE).

---

Built by [Mehmet Turaç](https://github.com/mturac). Pattern adaptations, additional language scaffolds, and stage cookbook entries welcome — see [CONTRIBUTING.md](./CONTRIBUTING.md).
