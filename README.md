# example-mcp-stateful

learning-first boilerplate for stateful MCP servers over Streamable HTTP with TypeScript SDK v2.

> part of a series: [stdio](https://github.com/yigitkonur/example-mcp-stdio) · [stateless](https://github.com/yigitkonur/example-mcp-stateless) · **stateful** (you are here) · [sse](https://github.com/yigitkonur/example-mcp-sse)

## what it does

- runs a stateful MCP server where sessions persist across requests with in-memory storage
- implements the full Streamable HTTP lifecycle: `POST /mcp` (JSON-RPC), `GET /mcp` (SSE stream), `DELETE /mcp` (session close)
- demonstrates SSE resumability via an in-memory event store with `Last-Event-Id` replay
- registers example tools (`add_note`, `list_notes`, `simulate_work`), resources (`session://overview`, `session://notes/{index}`), and a prompt (`study-notes`)
- includes a scaffold CLI to generate new projects from templates
- vendors MCP TypeScript SDK v2 pre-release tarballs for reproducible installs

## quick start

```bash
npm install
npm run dev
```

server starts at `http://127.0.0.1:1453`. endpoints:

- `POST /mcp` -- JSON-RPC requests
- `GET /mcp` -- SSE stream
- `DELETE /mcp` -- close session
- `GET /health` -- health check
- `GET /sessions` -- list active sessions

Docker alternative:

```bash
docker compose up --build -d
```

## scaffold cli

generate a new project from the included templates:

```bash
npm run create -- init my-mcp-app
```

options: `--force`, `--dir <path>`, `--sdk vendored|registry`. see [scaffold CLI docs](docs/03-scaffold-cli.md) for details.

## documentation

| doc | description |
|-----|-------------|
| [getting started](docs/01-getting-started.md) | install, run, verify endpoints |
| [architecture](docs/02-architecture.md) | module layout, session lifecycle, resumability |
| [scaffold cli](docs/03-scaffold-cli.md) | generate new projects from templates |
| [sdk v2 notes](docs/04-sdk-v2-notes.md) | v2 packages, vendoring, migration |
| [validation](docs/05-validation.md) | smoke tests, primitive checks, release checklist |

## sdk v2 context

this project targets MCP TypeScript SDK v2 pre-release (`2.0.0-alpha.0`). the v2 SDK splits the monolithic `@modelcontextprotocol/sdk` package into `server`, `node`, `client`, and `core`. this repo uses `@modelcontextprotocol/server` for `McpServer` and `@modelcontextprotocol/node` for `NodeStreamableHTTPServerTransport`. vendored tarballs under `vendor/mcp-sdk-v2/` ensure reproducible installs until v2 reaches stable release.

## license

MIT
