# MCP HTTP Stateful Boilerplate (TypeScript SDK v2)

Learning-first boilerplate for building **stateful MCP servers over Streamable HTTP** with official MCP TypeScript SDK v2 pre-release primitives.

This repository provides:

- a runnable reference server (`src/`)
- a scaffold creator CLI (`mcp-http-stateful-starter`)
- starter templates for generating new projects (`templates/http-stateful/`)

## Changelog (Latest First)

### 2026-02-21 - Major Rewrite for Upcoming TypeScript SDK v2

- Full migration from old v1-style architecture to v2 primitives.
- Switched to `@modelcontextprotocol/server` + `@modelcontextprotocol/node`.
- Rebuilt runtime around stateful Streamable HTTP (`POST`/`GET`/`DELETE /mcp`).
- Added scaffold creator CLI and starter templates.
- Added dedicated v2 docs and deep-dive notes.
- Kept installs reproducible with vendored official v2 pre-release tarballs.

Full details: `CHANGELOG.md`.

## About SDK v2 (Proper Context)

MCP TypeScript SDK v2 is currently pre-release on `main` and differs significantly from v1:

- package split: server/client/core (plus runtime adapters like node)
- server HTTP transport changed to `NodeStreamableHTTPServerTransport`
- handler and registration style centered on `registerTool` / `registerResource` / `registerPrompt`
- server-side SDK auth exports removed; auth should be external middleware
- server-side standalone SSE transport removed; Streamable HTTP is the migration path

Official references:

- SDK repository: <https://github.com/modelcontextprotocol/typescript-sdk>
- migration guide: <https://github.com/modelcontextprotocol/typescript-sdk/blob/main/docs/migration.md>
- server guide: <https://github.com/modelcontextprotocol/typescript-sdk/blob/main/docs/server.md>

## Quick Start

```bash
npm install
npm run dev
```

Endpoints:

- MCP: `http://127.0.0.1:1453/mcp`
- health: `http://127.0.0.1:1453/health`
- sessions: `http://127.0.0.1:1453/sessions`

## Scaffold Creator CLI

Create a new starter project:

```bash
npm run create -- init my-mcp-app
```

or from compiled output:

```bash
node dist/cli/index.js init my-mcp-app
```

Common options:

```bash
npm run create -- init my-mcp-app --force
npm run create -- init my-mcp-app --dir ./playground
npm run create -- init my-mcp-app --sdk registry
```

- default mode is `--sdk vendored` for reproducibility
- `--sdk registry` expects your environment to resolve v2 pre-release packages

## v2 Packaging Note

To keep this project reproducible while v2 is pre-release, this repo vendors official tarballs:

- `vendor/mcp-sdk-v2/modelcontextprotocol-server-2.0.0-alpha.0.tgz`
- `vendor/mcp-sdk-v2/modelcontextprotocol-node-2.0.0-alpha.0.tgz`

Pinned source commit: `c4ee360aac7afd2964785abac379a290d0c9847a` (SDK main branch, 2026-02-20).

## Docs

- docs index: `docs/README.md`
- getting started: `docs/getting-started.md`
- scaffold CLI guide: `docs/scaffold-cli.md`
- v2 SDK notes: `docs/v2-sdk-notes.md`
- protocol deep dive: `docs/http-stateful-v2-deep-dive.md`

## Validation Scripts

- `npm run check` - typecheck + lint + format check
- `npm run build` - compile TypeScript
- `npm run smoke` - end-to-end MCP smoke test
- `npm run ci` - check + build + smoke

## MCP CLI Verification

This server and scaffold output were verified using `mcp-cli`:

1. connect and inventory: `mcp-cli info <server>`
2. inspect tool schemas: `mcp-cli info <server> <tool>`
3. run valid and invalid tool calls: `mcp-cli call <server> <tool> '<json>'`
4. run with fresh sessions after rebuilds: `MCP_NO_DAEMON=1 ...`

`mcp-cli` is tool-focused, so resources/prompts/session lifecycle were additionally validated with direct JSON-RPC HTTP tests in a persistent session.

## License

MIT
