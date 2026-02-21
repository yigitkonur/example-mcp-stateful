# getting started

set up and run the stateful MCP HTTP server locally.

## prerequisites

- Node.js >= 20 (see `.nvmrc`)
- npm (ships with Node.js)
- Docker (optional, for containerized runs)

## install and run

### npm

```bash
npm install
npm run dev
```

`npm run dev` uses `tsx watch` for live reload during development. for production:

```bash
npm run build
npm run start
```

### Docker

```bash
docker compose up --build -d
```

the container exposes port 1453 and includes a health check against `/health`.

### Make

```bash
make install   # npm install
make dev       # npm run dev
make build     # npm run build
make run       # npm run start (built output)
make smoke     # npm run smoke
make docker-up # docker compose up --build -d
```

## default endpoints

| method   | path        | purpose                                      |
|----------|-------------|----------------------------------------------|
| `POST`   | `/mcp`      | JSON-RPC requests (initialize, tools/call, etc.) |
| `GET`    | `/mcp`      | open SSE stream for server-to-client messages |
| `DELETE` | `/mcp`      | close a session                               |
| `GET`    | `/health`   | health check (returns `{ ok: true }`)         |
| `GET`    | `/sessions` | list active sessions with metadata            |

all MCP endpoints default to `http://127.0.0.1:1453`.

## environment variables

configure via `.env` file or process environment. defaults are set for local development.

| variable              | default       | description                                        |
|-----------------------|---------------|----------------------------------------------------|
| `PORT`                | `1453`        | HTTP listen port                                   |
| `HOST`                | `127.0.0.1`  | bind address                                       |
| `CORS_ORIGIN`         | `*`           | allowed CORS origin                                |
| `SESSION_TTL_MS`      | `1800000`     | session inactivity timeout (30 min)                |
| `CLEANUP_INTERVAL_MS` | `60000`       | interval for expired session cleanup (60 sec)      |
| `EVENT_MAX_AGE_MS`    | `86400000`    | max age for stored SSE events (24 hours)           |
| `EVENT_MAX_COUNT`     | `5000`        | max number of SSE events retained in memory        |
| `ALLOWED_HOSTS`       | *(unset)*     | comma-separated hostnames for Host header validation. when unset, SDK localhost defaults apply |

see `.env.example` for a reference file.

## quick verify

after starting the server:

```bash
# health check
curl http://127.0.0.1:1453/health

# initialize a session
curl -X POST http://127.0.0.1:1453/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2025-03-26",
      "capabilities": {},
      "clientInfo": { "name": "curl-test", "version": "0.0.0" }
    }
  }'
```

the initialize response includes an `Mcp-Session-Id` header. use it for subsequent requests.

## next steps

- [architecture](02-architecture.md) -- understand the module layout and session lifecycle
- [scaffold CLI](03-scaffold-cli.md) -- generate a new project from this boilerplate
