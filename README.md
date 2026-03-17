# Agentic Data Stack Lite

The open-source stack for ClickHouse's suite of agentic analytic tools — your chat, your models, your data.
Powered by [ClickHouse](https://clickhouse.com), [LibreChat](https://librechat.ai), and [Langfuse](https://langfuse.com).

> Learn more at [clickhouse.ai](https://clickhouse.ai)

## Overview

This project runs a fully self-hosted agentic analytics environment with Docker Compose. It connects a chat UI (LibreChat) to your data (ClickHouse) via MCP — all in a single `docker compose up` command.

The stack is modular. Start lean and add components only when you need them.

### Baseline (default)

| Component | Purpose | Port |
|---|---|---|
| **ClickHouse** | World's fastest analytical database | `8123` |
| **ClickHouse MCP** | MCP server that gives agents access to ClickHouse | `8000` |
| **LibreChat** | Modern Chat UI with multi-model / provider support (OpenAI, Anthropic, Google) | `3080` |
| **MongoDB** | Database for LibreChat | `27017` |

### Optional: `--profile langfuse`

Adds LLM observability — traces, evals, and prompt management.

| Component | Purpose | Port |
|---|---|---|
| **Langfuse** | LLM observability UI | `3000` |
| **PostgreSQL** | Database for Langfuse | `5432` |
| **MinIO** | S3-compatible object storage for Langfuse | `9090` |
| **Redis** | Queue and cache for Langfuse | `6379` |

### Optional: `--profile rag`

Adds file uploads and retrieval-augmented generation.

| Component | Purpose | Port |
|---|---|---|
| **RAG API** | Retrieval-augmented generation service | `8001` |
| **pgvector** | Vector database for embeddings | `5433` |
| **Meilisearch** | Full-text search for LibreChat | `7700` |

## Quick Start

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose v2+

### 1. Prepare the environment

```bash
./scripts/prepare-demo.sh
```

This generates a `.env` file with random credentials for all services, then presents an interactive menu to optionally configure API keys for OpenAI, Anthropic, and/or Google. Any providers you skip will remain as `user_provided`, letting users enter their own keys in the LibreChat UI.

You can also generate credentials separately and customize the initial administrator account:

```bash
USER_EMAIL="you@example.com" USER_PASSWORD="supersecret" USER_NAME="YourName" ./scripts/generate-env.sh
```

### 2. Start the stack

**Baseline** — ClickHouse + MCP + LibreChat only:

```bash
docker compose up -d
```

**With observability** — adds Langfuse:

```bash
docker compose --profile langfuse up -d
```

**With file search / RAG**:

```bash
docker compose --profile rag up -d
```

> **Note:** RAG requires a real API key for embeddings — `user_provided` won't work. Set `RAG_OPENAI_API_KEY` in `.env` to a valid OpenAI key, or configure a different `EMBEDDINGS_PROVIDER`. See the [RAG API docs](https://librechat.ai/docs/configuration/rag_api) for details.

**Full stack** — everything:

```bash
docker compose --profile langfuse --profile rag up -d
```

### 3. Access the services

- **LibreChat** — [http://localhost:3080](http://localhost:3080)
- **Langfuse** — [http://localhost:3000](http://localhost:3000) *(if langfuse profile enabled)*
- **MinIO Console** — [http://localhost:9090](http://localhost:9090) *(if langfuse profile enabled)*

### 4. Log in

An admin account is created automatically on first startup. Your login credentials are in `.env`:

```bash
grep -E "LIBRECHAT_USER_EMAIL|LIBRECHAT_USER_PASSWORD" .env
```

Default values (if you used `prepare-demo.sh` without customizing):
- **Email:** `admin@admin.com`
- **Password:** `password`

To set your own credentials, pass them when generating the environment:

```bash
USER_EMAIL="you@example.com" USER_PASSWORD="supersecret" USER_NAME="YourName" ./scripts/generate-env.sh
```

> **Note:** Registration is disabled by default. The admin account is the only way in unless you create additional users via `./scripts/create-librechat-user.sh`.

### 5. Configure your LLM provider

No model provider is pre-configured — you bring your own API key. After logging in:

1. Click your profile icon → **Settings** → **API Keys**
2. Enter your API key for any supported provider (OpenAI, Anthropic, Google, etc.)
3. Your key is encrypted and stored securely; it is never shared

You can configure keys for multiple providers and switch between them per conversation.

## Architecture

![Architecture](assets/architecture.png)

LibreChat connects to ClickHouse through the MCP server, allowing AI agents to query and analyze your data. When Langfuse is enabled, all LLM interactions are traced for observability, evaluation, and prompt management.

## Scripts

| Script | Description |
|---|---|
| `scripts/prepare-demo.sh` | Generate `.env` and interactively configure API keys |
| `scripts/generate-env.sh` | Generate `.env` with random credentials |
| `scripts/reset-all.sh` | Stop all containers and wipe all data/volumes |
| `scripts/create-librechat-user.sh` | Manually create a LibreChat admin user |
| `scripts/init-librechat-user.sh` | Auto-init user on container startup (used internally) |

## Configuration

- **LibreChat** — `librechat.yaml` configures endpoints, MCP servers, and agent capabilities
- **Environment** — `.env` holds all credentials and service configuration (see `.env.example` for reference)
- **Docker** — `docker-compose.yml` includes five compose files:
  - `clickhouse-compose.yml` — ClickHouse database (baseline)
  - `clickhouse-mcp-compose.yml` — ClickHouse MCP server (baseline)
  - `librechat-compose.yml` — LibreChat and MongoDB (baseline)
  - `langfuse-compose.yml` — Langfuse, PostgreSQL, Redis, MinIO (`--profile langfuse`)
  - `rag-compose.yml` — RAG API, pgvector, Meilisearch (`--profile rag`)

## Reset Everything

To tear down all containers and delete all data:

```bash
./scripts/reset-all.sh
```

Then set up again and start fresh:

```bash
./scripts/prepare-demo.sh
docker compose up -d
```

## Links

- [clickhouse.ai](http://clickhouse.ai) — Project homepage
- [Documentation](https://clickhouse.com/docs/use-cases/AI/MCP/librechat) — Full setup guide for adding ClickHouse MCP to LibreChat
- [ClickHouse MCP](https://github.com/ClickHouse/mcp-clickhouse) — MCP server for ClickHouse
- [LibreChat](https://github.com/danny-avila/LibreChat) — Chat UI
- [Langfuse](https://langfuse.com) — LLM observability
