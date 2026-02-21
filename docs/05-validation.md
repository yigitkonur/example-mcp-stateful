# validation

verification procedures for the root server and generated projects.

## root validation

run the full CI pipeline locally:

```bash
npm run ci
```

this executes `check` (typecheck + lint + format check), `build`, and `smoke` in sequence.

individual steps:

```bash
npm run typecheck      # tsc --noEmit
npm run lint           # eslint
npm run format:check   # prettier --check
npm run build          # clean + compile
npm run smoke          # end-to-end smoke test against built output
```

the smoke test (`scripts/smoke.mjs`) starts the compiled server, sends an initialize request, calls a tool, checks health, deletes the session, and exits.

## generated project validation

after scaffolding a project:

```bash
npm run create -- init test-project
cd test-project
npm install
npm run dev
```

verify:

- server starts on port 1453
- `GET /health` returns `{ "ok": true }`
- `POST /mcp` with initialize returns a session ID
- subsequent requests with the session ID succeed

## mcp-cli verification

use `npx @anthropic-ai/mcp-cli` or any MCP-compatible client to interact with the running server.

### initialize

```bash
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
      "clientInfo": { "name": "test-client", "version": "0.0.0" }
    }
  }'
```

expected: SSE response with server capabilities and an `Mcp-Session-Id` response header.

### tools/list

```bash
curl -X POST http://127.0.0.1:1453/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Mcp-Session-Id: <session-id>" \
  -d '{ "jsonrpc": "2.0", "id": 2, "method": "tools/list" }'
```

expected: list containing `add_note`, `list_notes`, `simulate_work`.

### resources/list

```bash
curl -X POST http://127.0.0.1:1453/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Mcp-Session-Id: <session-id>" \
  -d '{ "jsonrpc": "2.0", "id": 3, "method": "resources/list" }'
```

expected: list containing `session-overview` and `session-note` resources.

### prompts/list

```bash
curl -X POST http://127.0.0.1:1453/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Mcp-Session-Id: <session-id>" \
  -d '{ "jsonrpc": "2.0", "id": 4, "method": "prompts/list" }'
```

expected: list containing `study-notes`.

### GET /mcp SSE stream

```bash
curl -N http://127.0.0.1:1453/mcp \
  -H "Mcp-Session-Id: <session-id>" \
  -H "Accept: text/event-stream"
```

expected: SSE stream stays open. server-to-client notifications (like log messages from `simulate_work`) appear as `data:` lines with event IDs.

### DELETE /mcp session close

```bash
curl -X DELETE http://127.0.0.1:1453/mcp \
  -H "Mcp-Session-Id: <session-id>"
```

expected: 200 response. subsequent requests with this session ID return 404.

## release checklist

- [ ] `npm run ci` passes (typecheck + lint + format + build + smoke)
- [ ] `npm run create -- init test-release` generates a valid project
- [ ] generated project installs and starts without errors
- [ ] Docker build succeeds: `docker compose up --build -d`
- [ ] Docker health check passes: `curl http://127.0.0.1:1453/health`
- [ ] all five endpoints respond correctly (POST, GET, DELETE `/mcp`, `/health`, `/sessions`)
- [ ] session lifecycle works end-to-end: initialize, tool call, SSE stream, delete
- [ ] GitHub Actions CI passes on the main branch
