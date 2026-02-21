# CLAUDE.md

Agent notes for this repository.

## Project purpose

This repo is a **learning-first MCP stateful HTTP boilerplate** based on the official TypeScript SDK v2 pre-release APIs.

It includes:

- a runnable example server (`src/`)
- a starter generator CLI (`src/cli/index.ts`)
- starter templates (`templates/http-stateful/`)

## Core runtime architecture

- `src/index.ts`: process entrypoint and graceful shutdown
- `src/http/createHttpApp.ts`: Express routes and transport/session wiring
- `src/mcp/createLearningServer.ts`: tools/resources/prompts registration
- `src/state/sessionRegistry.ts`: session runtime lifecycle
- `src/state/inMemoryEventStore.ts`: SSE resumability event storage
- `src/config.ts`: environment parsing and defaults

## MCP transport model

Only Streamable HTTP stateful mode is supported:

- `POST /mcp`
- `GET /mcp`
- `DELETE /mcp`

Transport implementation: `NodeStreamableHTTPServerTransport` from `@modelcontextprotocol/node`.

## SDK v2 pre-release packaging

This repo vendors official packed v2 artifacts under `vendor/mcp-sdk-v2/` for reproducible installs in environments where registry availability may vary.

## Development commands

- `npm run dev`
- `npm run check`
- `npm run build`
- `npm run smoke`
- `npm run ci`
- `npm run create -- init <name>`

## Cleanup policy

Keep the codebase lean:

- no v1 API usage/imports
- no dead code, duplicate initialization, or stale Redis-era docs
- no hidden side scripts; keep flows explicit in npm scripts and docs
