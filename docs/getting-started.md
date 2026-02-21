# Getting Started

## Prerequisites

- Node.js 20+
- npm

## Install and Run

```bash
npm install
npm run dev
```

## Endpoints

- `POST /mcp` - initialize and normal JSON-RPC requests
- `GET /mcp` - SSE stream endpoint (stateful transport)
- `DELETE /mcp` - terminate session
- `GET /health` - health status
- `GET /sessions` - active session summaries

Default base URL:

- `http://127.0.0.1:1453`

## Environment

See `.env.example`:

- `PORT`, `HOST`
- `SESSION_TTL_MS`, `CLEANUP_INTERVAL_MS`
- `EVENT_MAX_AGE_MS`, `EVENT_MAX_COUNT`
- `CORS_ORIGIN`
- `ALLOWED_HOSTS`

## Validate Locally

```bash
npm run check
npm run build
npm run smoke
```

or all at once:

```bash
npm run ci
```
