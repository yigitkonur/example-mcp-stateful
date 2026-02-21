# Docs - MCP HTTP Stateful Boilerplate (v2)

This documentation is rebuilt for the current v2 rewrite of the repository.

## Changelog (Latest First)

### 2026-02-21 - Major Rewrite for Upcoming TypeScript SDK v2

- Replaced legacy v1-era architecture and examples.
- Standardized around v2 server/node package split.
- Added scaffold creator CLI and template starter.
- Added reproducible vendored package strategy for pre-release v2.

Full changelog: `../CHANGELOG.md`.

## What Changed with v2

This project follows the modern v2 model:

- `@modelcontextprotocol/server` for server implementation
- `@modelcontextprotocol/node` for Node HTTP transport adapter
- `NodeStreamableHTTPServerTransport` for stateful Streamable HTTP
- registration APIs: `registerTool`, `registerResource`, `registerPrompt`

It intentionally avoids old v1 patterns from `@modelcontextprotocol/sdk`.

## Documentation Map

1. `docs/getting-started.md`
   Setup, run, endpoints, and local verification.
2. `docs/scaffold-cli.md`
   How to generate new projects using the scaffold creator CLI.
3. `docs/v2-sdk-notes.md`
   Practical notes and limitations for SDK v2 pre-release usage.
4. `docs/http-stateful-v2-deep-dive.md`
   Protocol-level details for stateful Streamable HTTP in this codebase.

## Current Validation Status

At the latest update, the following were re-verified:

- root project: `npm run ci` passed
- scaffold generation: CLI creates project structure successfully
- generated project: install + typecheck + build + health endpoint check passed
