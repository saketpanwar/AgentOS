# AgentOS — Where AI Specialists Collaborate and Ship

A multi-agent system where specialized AI agents — CEO, Planner, Developer, QA, Writer — work together like a high-performance engineering team to plan, build, and deliver outcomes from a single user goal.

**Live demo → [agentos.vercel.app](https://agentos.vercel.app)**

---

## What it does

You give AgentOS a goal. A coordinated team of AI agents takes it from there:

1. **CEO Agent** — interprets the goal and produces a structured execution plan
2. **Planner Agent** — decomposes the CEO plan into an ordered list of concrete steps
3. **Developer Agent** — writes working code, iterates on QA feedback
4. **QA Agent** — runs the code in a real headless browser, scores it, requests revisions
5. **Writer Agent** — generates documentation and a final deliverable report

Every step is streamed live to the UI via WebSockets. Six preset pipelines are included out of the box: Software Development, Research & Intelligence, Investment Analysis, Legal Review, Content Marketing, and Academic Literature Review. Custom pipelines can be built and saved in the UI.

---

## Architecture

```
Browser (Next.js 15)
    │  WebSocket + REST
    ▼
FastAPI Backend (Python 3.12)
    │  LangGraph orchestration
    ▼
┌─────────────────────────────────────────┐
│          Agent Orchestrator             │
│  CEO → Planner → Developer → QA → ...  │
└─────────────────────────────────────────┘
    │
    ├── Claude Opus 4.7 / Sonnet 4.6  (AI)
    ├── PostgreSQL on Neon             (persistence)
    ├── Redis on Upstash               (short-term memory)
    ├── Pinecone                       (vector memory)
    └── Cloudflare R2                  (generated artefacts)
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 15 (App Router), Tailwind CSS v4 |
| Backend | FastAPI, Python 3.12, Uvicorn |
| Orchestration | LangGraph |
| AI | Claude Opus 4.7 (reasoning), Claude Sonnet 4.6 (fast tasks) |
| Database | PostgreSQL via Neon |
| Cache / Memory | Redis via Upstash |
| Vector DB | Pinecone |
| File Storage | Cloudflare R2 |
| Deployment | Vercel (frontend) · Railway (backend) |

---

## Local Development

**Prerequisites:** Node 20+, Python 3.12

```bash
# Clone
git clone https://github.com/your-org/AgentOS.git
cd AgentOS

# Frontend
npm install
npm run dev        # http://localhost:3000

# Backend (in a separate terminal)
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # fill in API keys
uvicorn main:app --reload --port 8000
```

**Required env vars (`backend/.env`):**

```env
ANTHROPIC_API_KEY=sk-ant-...
DATABASE_URL=postgresql://...
DATABASE_URL_UNPOOLED=postgresql://...
```

**Optional env vars:**

```env
UPSTASH_REDIS_URL=https://...     # short-term agent memory
UPSTASH_REDIS_TOKEN=...
PINECONE_API_KEY=...              # long-term vector memory
PINECONE_INDEX=agentos-memory
R2_ACCOUNT_ID=...                 # file storage for generated artefacts
R2_ACCESS_KEY_ID=...
R2_SECRET_ACCESS_KEY=...
R2_BUCKET_NAME=agentos-artefacts
SERPER_API_KEY=...                # web search (serper.dev)
BRAINTRUST_API_KEY=...            # LLM eval tracking
JWT_SECRET=...                    # auth (change in production)
DEV_MODE=true                     # collapses all models to Haiku to save cost
```

---

## Design Decisions

**LangGraph over a custom orchestrator** — models agent workflows as directed graphs with typed state, making complex branching (QA revision loops, conditional routing) explicit and easy to reason about.

**Claude Opus 4.7 + Sonnet 4.6** — Opus 4.7 for deep reasoning and code generation; Sonnet 4.6 for fast, lighter sub-tasks like QA scoring and content editing. Swapping models is a one-line config change.

**QA runs real code** — the QA agent spins up a headless Chromium browser (Playwright), loads generated HTML/JS files from R2, and runs scripted test steps. It doesn't just read the source; it actually executes the app.

**PostgreSQL (Neon) over SQLite** — serverless Postgres with zero cold-start. Stores tasks, subtasks, messages, metrics, and custom presets. Redis and Pinecone degrade gracefully if not configured.

**WebSockets over SSE** — agent runs span minutes with many sub-events. A persistent WebSocket connection keeps the UI updated without polling.
