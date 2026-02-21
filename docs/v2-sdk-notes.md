# SDK v2 Notes (Practical)

## Current State

This project targets MCP TypeScript SDK v2 pre-release APIs on the `main` branch of the official SDK repository.

## Key v2 Shifts Used Here

- package split (server/client/core model)
- Node HTTP transport via `@modelcontextprotocol/node`
- `NodeStreamableHTTPServerTransport` in stateful mode
- registration APIs (`registerTool`, `registerResource`, `registerPrompt`)

## What Was Intentionally Removed vs Older Patterns

- no usage of `@modelcontextprotocol/sdk` monolithic imports
- no server-side SDK auth helpers
- no server SSE transport implementation path

## Pre-release Limitation Handling

In some environments, npm resolution for v2 split packages may vary.

This repo handles that by vendoring official packed artifacts and wiring dependencies to those tarballs by default.

## Official References

- SDK repo: <https://github.com/modelcontextprotocol/typescript-sdk>
- migration guide: <https://github.com/modelcontextprotocol/typescript-sdk/blob/main/docs/migration.md>
- server guide: <https://github.com/modelcontextprotocol/typescript-sdk/blob/main/docs/server.md>
