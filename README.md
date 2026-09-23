# Second Brain

Personal knowledge system driven by a Telegram bot. Save notes, links, photos, and documents; they get archived, OCR'd, embedded, and become searchable through an AI agent with memory and reminders.

## Stack

- Node.js + TypeScript (tsx)
- Hono
- Prisma 7.9 + PostgreSQL 16 (metadata only — vectors live in Qdrant)
- Redis + BullMQ (embedding + reminder pipelines)
- Qdrant (vector store, cosine, 2048-dim)
- Mistral OCR (document/photo text extraction)
- Cloudflare R2 (file archive)
- OpenRouter embeddings (via `/usr/bin/curl` — bypasses Cloudflare bot protection)
- OpenAI-compatible chat model for the agent (`@anvia/core`)
- React + Vite dashboard (served by the API at `/dashboard`)
- MCP server (`@modelcontextprotocol/server`, stdio)
- pnpm workspace

## Structure

```text
.
├── apps/
│   ├── api/               # Hono HTTP API, admin dashboard mount, BullMQ embedding worker
│   │   ├── prisma/        # schema + migrations
│   │   └── src/
│   │       ├── index.ts   # HTTP API (health, memory CRUD, URL scrape, search, /dashboard)
│   │       ├── admin.ts   # admin dashboard app (mounted at /dashboard)
│   │       ├── worker.ts  # BullMQ worker: download → OCR → R2 → embed → Qdrant
│   │       ├── embed.ts   # OpenRouter embeddings (curl-based)
│   │       ├── vector.ts  # Qdrant collection ops
│   │       ├── r2.ts      # R2 archiver
│   │       └── queue.ts   # BullMQ embedding queue
│   ├── platform/          # Telegram bot: agent runtime, access control, reminders
│   │   └── src/
│   │       ├── index.ts   # bot wiring: agent, memory store, MCP client, Lens
│   │       ├── bot.ts     # Telegram long-polling bot
│   │       ├── handler.ts # message/callback handling (save, search, reminders)
│   │       ├── access.ts / access-store.ts  # admin approve/deny users
│   │       ├── memory-store.ts  # Prisma-backed MemoryStore
│   │       ├── reminder-worker.ts  # BullMQ reminder delivery worker
│   │       ├── eval.ts / cli.ts   # evals + agent CLI
│   │       └── telegram.test.ts / handler.test.ts / bot.test.ts  # vitest
│   ├── dashboard/         # React + Vite admin UI (build served at /dashboard)
│   └── mcp-server/        # MCP stdio server exposing memory tools + reminders
└── packages/
    └── agent/             # @second-brain/agent: intent detection, prompts, guardrails,
                            # agent tools (save/search/reminder), conversation memory,
                            # reminder queue, tracing
```

## Requirements

- Node.js 22, pnpm 10
- Docker Compose, or reachable PostgreSQL / Redis / Qdrant

## Setup

```bash
cp .env.example .env   # fill in values
pnpm install           # generates Prisma client into apps/api/src/generated/
```

Postgres needs database `secondbrain`. Schema: users/access control (`User`, `AccessRequest`), memories (`Memory`), reminders (`Reminder`).

## Infrastructure

```bash
docker compose up -d   # postgres, redis, qdrant + app services (migrate, api, worker, reminder)
```

Run only the backing services with:

```bash
docker compose up -d postgres redis qdrant
```

## Prisma

```bash
pnpm --dir apps/api run db:generate   # regenerate client (after schema edits)
pnpm --dir apps/api run db:migrate -- --name <name>
pnpm --dir apps/api run db:studio
```

All prisma commands load the root `.env` via `dotenv-cli`. `apps/api/src/generated/` is gitignored — run `db:generate` after clone.

## Run

```bash
pnpm --dir apps/api run dev        # HTTP API + dashboard on :3000
pnpm --dir apps/api run worker     # embedding pipeline worker
pnpm --dir apps/platform run dev   # Telegram agent bot (long polling)
pnpm --dir apps/platform run reminder-worker  # reminder delivery worker
pnpm --dir apps/dashboard run dev  # dashboard dev server (Vite)
pnpm --dir apps/mcp-server run dev # MCP server (stdio)
```

## Data flow

1. User sends note/link/photo/document to Telegram bot.
2. Access control: new users start `PENDING`; admin approves/denies via the bot or `/dashboard`.
3. Bot persists `Memory` row and enqueues an embedding job (explicit save phrases like "simpan"/"remember this" or agent-detected intent). URL saves get scraped server-side.
4. Worker: downloads Telegram file → Mistral OCR → archives to R2 → embeds via OpenRouter → upserts vector in Qdrant.
5. Search: bot embeds the query, searches Qdrant filtered by `userId`, agent answers from results in the user's language.
6. Reminders: agent tool schedules a delayed BullMQ job; `reminder-worker` delivers via Telegram.
7. The same memory tools are exposed over MCP for external clients.

## Environment

See `.env.example` for the full list. Never commit `.env` — it's gitignored.

| Var | Purpose |
| --- | --- |
| `DATABASE_URL` | PostgreSQL connection |
| `REDIS_URL` | Redis for BullMQ |
| `QDRANT_URL` | Qdrant instance |
| `EMBED_URL` / `EMBED_MODEL` / `EMBED_API_KEY` | OpenRouter embeddings |
| `OPENAI_BASE_URL` / `OPENAI_API_KEY` / `AGENT_MODEL` | Chat model for the agent |
| `JUDGE_MODEL` | Model for evals |
| `TELEGRAM_BOT_TOKEN` / `ADMIN_TELEGRAM_ID` | Bot + admin approval |
| `DASHBOARD_URL` | Dashboard URL sent in `/dashboard` bot message |
| `API_PORT` | Host port for API + dashboard (default 3000) |
| `MISTRAL_API_KEY` | Mistral OCR |
| `R2_*` | Cloudflare R2 archive |
| `ANVIA_LENS_*` | Anvia Lens observability (empty = disabled) |
| `MCP_SERVER_COMMAND` | MCP server launch command (default: tsx mcp-server) |

## Security Considerations

We strive to maintain a secure codebase. All sensitive credentials are removed from Git history, and dependencies are regularly audited for known vulnerabilities. Please report any security concerns by opening an issue.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Tests

```bash
pnpm --dir packages/agent test     # vitest
pnpm --dir apps/platform test      # vitest
```

## Deploy

Docker image builds the dashboard, generates the Prisma client, and runs services from `docker-compose.yaml`: `migrate` (one-shot), `api`, `worker` (embeddings), `reminder` (delivery).
