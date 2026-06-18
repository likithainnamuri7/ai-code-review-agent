# AI Code Review Agent

A production-style automated code review service that analyzes GitHub repositories
and pull requests using AI — returning structured findings, scores, and inline suggestions.

## What It Does

Paste any GitHub repo or PR URL → the agent fetches the diff, runs parallel
analysis across 6 categories, and returns a scored review with actionable findings.

## Features

- **6 parallel analyzers** — bug detection, security, performance, style, complexity, best practices
- **Hybrid analysis pipeline** — regex + AST + Groq LLM (via LiteLLM)
- **RAG-enhanced reviews** — ChromaDB vector store for context-aware suggestions
- **0–100 scoring engine** — severity-weighted verdict per file and overall
- **GitHub integration** — reviews latest commit or a specific PR number
- **Inline comment posting** — posts findings directly as GitHub PR comments
- **Semantic caching** — Redis-based cache to avoid redundant LLM calls
- **Graceful degradation** — falls back to rule-based analysis if LLM is unavailable

## Tech Stack

- **Backend**: Python, FastAPI
- **AI Orchestration**: LangGraph, LiteLLM, Groq API
- **Vector Store**: ChromaDB
- **Cache**: Redis
- **Database**: PostgreSQL (asyncpg)
- **Observability**: OpenTelemetry

## Project Structure
src/

main.py                  # FastAPI entry point

api/

webhook.py             # POST /review, POST /webhook/github

orchestration/

graph.py               # Core LangGraph review engine

analysis/

bug_detection.py

security.py

performance.py

style.py

complexity.py

best_practices.py

llm_analyzer.py        # Groq LLM-based analyzer

ingestion/

diff.py                # Unified diff parser

rag/

vector_store.py        # ChromaDB + embeddings

output/

scoring.py             # Score aggregation

github_comment.py      # PR comment posting

cache/

semantic_cache.py      # Redis semantic cache

observability/           # OpenTelemetry tracing

## Setup

### 1. Clone & Install

```bash
git clone https://github.com/likithainnamuri7/Production-code-review-agent
cd Production-code-review-agent
pip install -r requirements.txt
```

### 2. Environment Variables

Copy `.env.example` to `.env` and fill in:


### 3. Run

```bash
uvicorn src.main:app --reload --port 8000
```

Open `http://localhost:8000`

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/review` | Submit repo/PR for review |
| `POST` | `/webhook/github` | GitHub webhook receiver |
| `GET`  | `/health` | Health check |

### Request Example

```json
POST /review
{
  "repo": "owner/repo",
  "pr_number": 42
}
```

Set `pr_number: 0` to review the latest commit on the default branch.

## Review Flow

1. User pastes GitHub URL → frontend extracts `owner/repo`
2. Backend fetches diff from GitHub API
3. Diff parsed into hunks → routed to 6 parallel analyzers
4. Rule-based + LLM analysis runs concurrently
5. Results aggregated → 0–100 score computed
6. Frontend renders score ring, findings, and file links

## Notes

- Accepts bare repo URLs (reviews latest commit) or specific PR URLs
- File links resolve via `sha → branch → baseBranch → main` fallback to avoid 404s
- LLM analysis uses Groq (fast inference); falls back to regex/AST if unavailable
