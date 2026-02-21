# HTTP Stateful v2 Deep Dive

This document explains the stateful Streamable HTTP behavior implemented in this repository.

## Changelog Context

The current behavior reflects the **2026-02-21 major rewrite for SDK v2**.

## Transport Model in This Project

- transport: `NodeStreamableHTTPServerTransport`
- mode: stateful (`sessionIdGenerator` is set)
- session lifecycle mapped to:
  - `POST /mcp`
  - `GET /mcp`
  - `DELETE /mcp`

## Session Lifecycle

1. Client sends `initialize` via `POST /mcp` without `Mcp-Session-Id`.
2. Server creates runtime state and transport.
3. Response includes `Mcp-Session-Id` header.
4. Client reuses the same session ID for subsequent POST/GET/DELETE requests.

## Resumability

This server enables resumability with an in-memory `EventStore`.

- SSE events are stored with event IDs.
- reconnecting GET requests can send `Last-Event-Id`.
- server replays missed events after the given event ID.

## Why In-Memory Here

This is a learning boilerplate, so stores are intentionally in-memory and simple.

For production, replace session and event storage with durable systems (e.g., Redis-backed implementations).

## Request Expectations

### POST `/mcp`

- JSON-RPC body
- initialize without session header for first request
- session header required for follow-up requests

### GET `/mcp`

- opens SSE channel for server-to-client stream behavior
- same `Mcp-Session-Id` required
- optional `Last-Event-Id` for replay

### DELETE `/mcp`

- terminates active session

## Related Code

- `src/http/createHttpApp.ts`
- `src/state/sessionRegistry.ts`
- `src/state/inMemoryEventStore.ts`
- `src/mcp/createLearningServer.ts`
