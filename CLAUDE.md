# CLAUDE.md

## project

learning-first boilerplate for stateful MCP servers over Streamable HTTP, built on TypeScript SDK v2 pre-release.

## what's inside

- `src/index.ts` -- process entrypoint, HTTP server, graceful shutdown
- `src/config.ts` -- Zod-based environment variable parsing
- `src/http/createHttpApp.ts` -- Express routes, transport/session wiring, cleanup timer
- `src/mcp/createLearningServer.ts` -- MCP tools/resources/prompts registration
- `src/state/sessionRegistry.ts` -- session runtime lifecycle (store, lookup, expire, close)
- `src/state/inMemoryEventStore.ts` -- in-memory EventStore for SSE resumability
- `src/cli/index.ts` -- scaffold CLI with `init` command
- `templates/http-stateful/` -- template files for generated projects
- `vendor/mcp-sdk-v2/` -- vendored SDK v2 pre-release tarballs
- `scripts/smoke.mjs` -- end-to-end smoke test

## transport

stateful Streamable HTTP. three MCP endpoints on `/mcp`:

- `POST /mcp` -- JSON-RPC requests (initialize creates session, subsequent requests reuse it)
- `GET /mcp` -- SSE stream for server-to-client notifications
- `DELETE /mcp` -- close session and clean up runtime

sessions are identified by the `Mcp-Session-Id` header. in-memory session registry with TTL-based expiration.

## SDK rules

- use only v2 APIs: `McpServer`, `registerTool`, `registerResource`, `registerPrompt`
- no v1 imports (`@modelcontextprotocol/sdk`, `SSEServerTransport`, `server.tool()`)
- Zod v4 (`zod/v4`) for all schemas
- vendored tarballs in `vendor/mcp-sdk-v2/` -- do not delete unless switching to registry
- `@modelcontextprotocol/server` for server logic, `@modelcontextprotocol/node` for Node.js HTTP transport

## commands

- `npm run dev` -- start with live reload (tsx watch)
- `npm run build` -- clean + compile TypeScript
- `npm run start` -- run compiled output
- `npm run check` -- typecheck + lint + format check
- `npm run smoke` -- end-to-end smoke test
- `npm run ci` -- check + build + smoke
- `npm run create -- init <name>` -- scaffold a new project
- `make docker-up` / `make docker-down` -- Docker Compose lifecycle
