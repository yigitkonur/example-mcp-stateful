reference implementation of a stateful MCP server over streamable HTTP. sessions persist across requests via in-memory or Redis-backed storage. includes event sourcing for SSE resumability and just-in-time instance reconstruction for horizontal scaling without sticky sessions.

ships as a calculator service — arithmetic, scientific math, history, prompts — but the point is the infrastructure patterns, not the math.

```bash
npm install && npm run dev
# listening on :1453
```

[![node](https://img.shields.io/badge/node-20+-93450a.svg?style=flat-square)](https://nodejs.org/)
[![typescript](https://img.shields.io/badge/typescript-5.7-93450a.svg?style=flat-square)](https://www.typescriptlang.org/)
[![license](https://img.shields.io/badge/license-MIT-grey.svg?style=flat-square)](https://opensource.org/licenses/MIT)

> part of a series: [stdio](https://github.com/yigitkonur/example-mcp-server-stdio) · [stateless HTTP](https://github.com/yigitkonur/example-mcp-server-http-stateless) · **stateful HTTP** (you are here)

---

## what it does

single `/mcp` endpoint handles the full MCP lifecycle:

- `POST /mcp` — JSON-RPC commands. initializes sessions (via `mcp-session-id` header) and handles all tool/resource/prompt calls
- `GET /mcp` — SSE stream for server-to-client push. supports `Last-Event-Id` for resumability
- `DELETE /mcp` — explicit session teardown
- `GET /health` — health check with storage mode, session count, uptime
- `GET /metrics` — Prometheus scrape endpoint (`mcp_calculations_total`, `mcp_active_sessions`)

### tools

| tool | what it does |
|:---|:---|
| `calculate` | basic arithmetic (`add`, `subtract`, `multiply`, `divide`). optional streaming progress |
| `batch_calculate` | multiple operations in one call with incremental progress |
| `advanced_calculate` | `factorial`, `power`, `sqrt`, `log`, `sin`, `cos`, `tan` |
| `demo_progress` | pure progress notification demo, no state changes |

### resources

| resource | URI |
|:---|:---|
| math constants | `calculator://constants` — pi, e, sqrt2, ln2, ln10 |
| calculation history | `calculator://history/{id}` — per-session, ring buffer of 50 |
| stats | `calculator://stats` — live from Prometheus registry |
| session info | `session://info/{id}` — uptime, request count, last activity |
| formulas | `formulas://library` — quadratic, pythagorean, compound interest |

### prompts

`explain-calculation`, `generate-problems`, `solve_math_problem`, `explain_formula`, `calculator_assistant` — all generate user-facing messages for LLM consumption.

## install

### local (in-memory, no dependencies)

```bash
npm install
npm run dev     # tsx watch, hot-reload
```

### with Redis

```bash
docker compose up --build -d
# or: make up
```

### production with secrets

```bash
echo "your-password" > redis_password.txt
make prod-secure-up
```

## configuration

all via environment variables, no config files loaded at runtime.

| variable | default | description |
|:---|:---|:---|
| `PORT` | `1453` | listen port |
| `USE_REDIS` | `false` | `true` enables Redis-backed sessions and event sourcing |
| `REDIS_HOST` | `localhost` | Redis host |
| `REDIS_PORT` | `6379` | Redis port |
| `REDIS_PASSWORD` | — | Redis auth (optional) |
| `REDIS_DB` | `0` | Redis database index |
| `SESSION_TIMEOUT` | `1800000` | session TTL in ms (30 min) |
| `RATE_LIMIT_WINDOW` | `900000` | rate limit window in ms (15 min) |
| `RATE_LIMIT_MAX` | `1000` | max requests per window |
| `CORS_ORIGIN` | `*` | allowed origin |
| `SAMPLE_TOOL_NAME` | — | if set, registers an extra echo tool with this name |
| `LOG_LEVEL` | `info` | log level |

## architecture

two files do all the work:

```
src/
  types.ts    — interfaces, Zod schemas, custom error hierarchy
  server.ts   — storage implementations, MCP registration, Express app, lifecycle
```

### storage layer (strategy pattern)

`ISessionStore` interface with two implementations:

- **`InMemorySessionStore`** — `Map<string, SessionData>` with manual expiry check on read and periodic cleanup
- **`RedisSessionStore`** — `mcp_session:{id}` keys with JSON serialization and TTL. fail-safe reads, fail-loud writes

same pattern for events:

- **`InMemoryEventStore`** — 10k event ring buffer, 24h max age, sequential ID generation
- **`RedisEventStore`** — Redis Streams with `XADD MAXLEN ~ 10000`, `XREAD` for replay

### session lifecycle

1. `POST /mcp` with no session header + `initialize` body → server generates UUID, stores `SessionData` to persistent store first, then creates `McpServer` and connects transport
2. subsequent requests carry `mcp-session-id` header → looked up in local cache or reconstructed from store (JIT reconstruction)
3. `DELETE /mcp` → transport closed, removed from cache, deleted from store, Prometheus gauge decremented

### JIT instance reconstruction

when a request hits a node that doesn't have the session in its local `Map`, the server:

1. verifies session exists in persistent store
2. reconstructs a new `StreamableHTTPServerTransport` with the existing session ID
3. creates a fresh `McpServer`, connects, caches locally

this is what makes horizontal scaling work without sticky sessions or shared memory.

### event sourcing / SSE resumability

both event store implementations support `replayEventsAfter(lastEventId)`. when a client reconnects with `Last-Event-Id`, all missed events are replayed through the SSE stream.

## docker

multi-stage Dockerfile: `node:22-alpine` builder → production image with `dumb-init` (proper PID 1 signal handling), non-root `nodejs` user (UID 1001), prod-only deps. built-in `HEALTHCHECK` on `/health`.

```yaml
# docker-compose.yml brings up mcp-server + redis:7-alpine
# docker-compose.prod.yml adds persistence (appendonly) and named volumes
# docker-compose.secrets.yml adds Docker Secrets for REDIS_PASSWORD
```

## metrics

| metric | type | description |
|:---|:---|:---|
| `mcp_calculations_total` | counter | labeled by `operation` (add, subtract, multiply, ...) |
| `mcp_active_sessions` | gauge | current session count |

scrape at `GET /metrics`.

## project structure

```
src/
  types.ts                    — data contracts, Zod schemas, error hierarchy
  server.ts                   — everything else (storage, MCP, Express, lifecycle)
docker-compose.yml            — base: server + Redis
docker-compose.dev.yml        — hot-reload, debug port 9229
docker-compose.prod.yml       — Redis persistence, named volumes
docker-compose.secrets.yml    — Docker Secrets for Redis password
Dockerfile                    — multi-stage, non-root, dumb-init
Makefile                      — convenience targets
smithery.yaml                 — Smithery registry metadata
.github/workflows/ci.yml      — typecheck + lint + format + build
```

## license

MIT
