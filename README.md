# recsys-pipeline-architect

A Claude skill for designing composable recommendation, ranking, and feed pipelines.

The skill encodes the six-stage pattern popularized by xAI's open-sourced [X For You algorithm](https://github.com/xai-org/x-algorithm) (Apache 2.0) — and applies it to use cases beyond social feeds: content CMSs, task prioritizers, RAG rerankers, notification triage, anywhere you need to pick "the top K for this user/context."

## What's in the box

- **`SKILL.md`** — the main skill. Drop into `~/.claude/skills/recsys-pipeline-architect/` for Claude Code, or copy into a Claude.ai project as instructions.
- **`references/`** — load-on-demand deep dives: interfaces in 4 languages, multi-action scoring pattern, candidate isolation, filter and scorer cookbooks.
- **`examples/`** — three runnable scaffolds in different stacks.

## The pattern

Every pipeline this skill produces has six stages, in order:

| # | Stage | Job | Parallel? |
|---|---|---|---|
| 1 | Source | Fetch candidates from one or more origins | Yes |
| 2 | Hydrator | Enrich candidates with metadata | Yes |
| 3 | Filter | Drop ineligible candidates | No (sequential) |
| 4 | Scorer | Assign scores | No (chain order matters) |
| 5 | Selector | Sort and pick top K | Single op |
| 6 | SideEffect | Async post-actions | Async (non-blocking) |

The skill walks you through each stage, surfaces the trade-offs (multi-action vs single-score, candidate isolation vs joint scoring, online vs offline), and generates a runnable scaffold in your stack.

## Examples

### Strapi v5 content feed (TypeScript)

`examples/strapi-content-feed/` — a Strapi plugin that adds `GET /api/feed/for-you` to any Strapi instance with an `article` content type. Multi-action scoring with `P(read)`, `P(like)`, `P(share)`, `P(skip)`. Author diversity. Standard filters.

### Zentra-compatible pipeline (Go)

`examples/zentra-go/` — Go implementation packaged as a Zentra `engine.Module`. The `pipeline/` package is standalone and usable outside Zentra.

### PMAI task prioritizer (Python / FastAPI)

`examples/pmai-task-prioritizer/` — applies the pattern to task ranking. `GET /tasks/next?user_id=42&limit=10` returns the top tasks for a user based on priority, due date, in-progress status, and project diversity. Verified runnable: includes pytest suite and a working FastAPI endpoint.

## Installing the skill

### Claude Code

```bash
mkdir -p ~/.claude/skills/
git clone https://github.com/<your-username>/recsys-pipeline-architect.git \
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

(Integrating into a real Strapi v5 project: see `examples/strapi-content-feed/README.md`.)

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

MIT. See [LICENSE](./LICENSE).
