# Changelog

## 2026-02-21 - Major Rewrite for Upcoming TypeScript SDK v2

- Replaced legacy v1-style implementation with a full v2-oriented architecture.
- Migrated to `@modelcontextprotocol/server` + `@modelcontextprotocol/node` primitives.
- Rewrote runtime around stateful Streamable HTTP lifecycle (`POST`/`GET`/`DELETE /mcp`).
- Added reusable starter CLI (`mcp-http-stateful-starter`) and scaffold templates.
- Removed old Redis-focused and calculator-specific monolith (`src/server.ts`, `src/types.ts`).
- Added learning deep-dive docs for v2 transport behavior and limitations.
- Added vendored official v2 pre-release package artifacts for reproducible installs.
- Tightened CI to run check + build + smoke test.
