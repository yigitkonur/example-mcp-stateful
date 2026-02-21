# architecture

module layout, session lifecycle, and design rationale for the stateful MCP HTTP server.

## module layout

| file                              | purpose                                                    |
|-----------------------------------|------------------------------------------------------------|
| `src/index.ts`                    | process entrypoint, creates HTTP server, handles graceful shutdown (SIGINT/SIGTERM) |
| `src/config.ts`                   | parses environment variables with Zod, exports `AppConfig` |
| `src/http/createHttpApp.ts`       | Express app with POST/GET/DELETE `/mcp` routes, `/health`, `/sessions`; wires transport, session registry, and cleanup timer |
| `src/mcp/createLearningServer.ts` | creates `McpServer` and registers tools, resources, and prompts |
| `src/state/sessionRegistry.ts`    | manages session runtime lifecycle: store, lookup, touch, expire, close |
| `src/state/inMemoryEventStore.ts` | in-memory `EventStore` implementation for SSE resumability |

supporting files:

| file/directory                    | purpose                                        |
|-----------------------------------|------------------------------------------------|
| `src/cli/index.ts`               | scaffold CLI (`init` command)                  |
| `templates/http-stateful/`       | template files for generated projects          |
| `vendor/mcp-sdk-v2/`             | vendored SDK v2 pre-release tarballs           |
| `scripts/smoke.mjs`              | end-to-end smoke test                          |

## session lifecycle

1. **client sends `initialize`** -- a `POST /mcp` without an `Mcp-Session-Id` header, containing an `initialize` JSON-RPC request
2. **server creates session** -- allocates a `SessionModel` (UUID, timestamps, empty notes), an `InMemoryEventStore`, an `McpServer`, and a `NodeStreamableHTTPServerTransport`; stores all of this as a `SessionRuntime` in the `SessionRegistry`
3. **server returns session ID** -- the response includes an `Mcp-Session-Id` header with the UUID
4. **client reuses session** -- subsequent `POST /mcp` requests include the `Mcp-Session-Id` header; the server looks up the existing runtime and delegates to its transport
5. **client opens SSE stream** -- `GET /mcp` with the session ID opens a server-sent events stream for server-to-client notifications
6. **client closes session** -- `DELETE /mcp` with the session ID triggers transport close and registry cleanup
7. **server expires idle sessions** -- a background timer runs every `CLEANUP_INTERVAL_MS` and closes sessions that have been inactive longer than `SESSION_TTL_MS`

## resumability model

the server uses `InMemoryEventStore` to support SSE stream resumability:

- every outgoing SSE message is stored with a monotonically increasing event ID (`{timestamp}-{sequence}`)
- events are indexed by stream ID for efficient replay
- when a client reconnects with `Last-Event-Id`, the transport calls `replayEventsAfter()` to send all events after that cursor
- old events are pruned by both age (`EVENT_MAX_AGE_MS`) and count (`EVENT_MAX_COUNT`)

this is an in-memory store. if the server restarts, event history is lost. for production use, replace with a durable store (Redis, database, etc.).

## endpoint contract

| method   | path        | headers required           | request body          | response                          |
|----------|-------------|----------------------------|-----------------------|-----------------------------------|
| `POST`   | `/mcp`      | none (initialize) or `Mcp-Session-Id` | JSON-RPC request/batch | SSE stream with JSON-RPC response |
| `GET`    | `/mcp`      | `Mcp-Session-Id`           | none                  | SSE stream (server notifications) |
| `DELETE` | `/mcp`      | `Mcp-Session-Id`           | none                  | 200 on success                    |
| `GET`    | `/health`   | none                       | none                  | `{ ok, activeSessions, mode, sdk }` |
| `GET`    | `/sessions` | none                       | none                  | `{ activeSessions, sessions[] }`  |

error responses use JSON-RPC error format: `{ jsonrpc: "2.0", error: { code, message }, id: null }`.

## registered MCP primitives

### tools

| name             | description                                | input                                    |
|------------------|--------------------------------------------|------------------------------------------|
| `add_note`       | stores a note in the current session       | `{ note: string }`                       |
| `list_notes`     | returns all notes in the current session   | `{}`                                     |
| `simulate_work`  | sends periodic log notifications to demonstrate streaming | `{ steps, delayMs, closeSseAfterStep? }` |

### resources

| name               | URI pattern              | description                     |
|--------------------|--------------------------|---------------------------------|
| `session-overview` | `session://overview`     | session metadata as JSON        |
| `session-note`     | `session://notes/{index}`| read one note by index          |

### prompts

| name          | description                              | args                |
|---------------|------------------------------------------|---------------------|
| `study-notes` | creates a prompt from session notes      | `{ objective? }`    |

## design rationale

this is a **learning-first boilerplate**. the design choices optimize for clarity:

- **in-memory session storage** -- no external dependencies needed to run. replace `SessionRegistry` with a persistent store for production.
- **in-memory event store** -- demonstrates SSE resumability mechanics without requiring Redis. not durable across restarts.
- **single-process model** -- all sessions live in one process. horizontal scaling requires sticky sessions or a shared session store.
- **Zod config parsing** -- validates environment variables at startup with clear error messages.
- **explicit wiring** -- no dependency injection framework. each module is a plain function that receives its dependencies.

## next steps

- [scaffold CLI](03-scaffold-cli.md) -- generate a project from this boilerplate
- [SDK v2 notes](04-sdk-v2-notes.md) -- understand the v2 package split and vendoring
