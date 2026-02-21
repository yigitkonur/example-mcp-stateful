# SDK v2 notes

context for the MCP TypeScript SDK v2 pre-release packages used in this project.

## v2 packages used

| package                          | version           | purpose                                   |
|----------------------------------|-------------------|-------------------------------------------|
| `@modelcontextprotocol/server`   | `2.0.0-alpha.0`  | `McpServer`, `ResourceTemplate`, schema types, `EventStore` interface, host validation utilities |
| `@modelcontextprotocol/node`     | `2.0.0-alpha.0`  | `NodeStreamableHTTPServerTransport` for Express integration |

both are installed from vendored tarballs under `vendor/mcp-sdk-v2/`.

## v1 patterns removed

the v2 SDK makes several breaking changes from v1. this project uses only v2 APIs:

| v1 pattern                        | v2 replacement                                    |
|-----------------------------------|---------------------------------------------------|
| `@modelcontextprotocol/sdk`      | split into `server`, `node`, `client`, `core`     |
| `SSEServerTransport`             | `NodeStreamableHTTPServerTransport`               |
| `server.tool()` / `server.resource()` | `server.registerTool()` / `server.registerResource()` |
| `server.prompt()`                | `server.registerPrompt()`                         |
| server-side SDK auth exports     | removed; use external middleware                  |
| Zod v3 schemas                   | Zod v4 (`zod/v4`) for tool input/output schemas  |

## vendoring strategy

v2 is pre-release (`2.0.0-alpha.0`). to ensure reproducible installs:

1. official tarballs are packed from the SDK repo (`npm pack`) at a pinned commit
2. tarballs are stored in `vendor/mcp-sdk-v2/`
3. `package.json` references them via `file:` protocol

pinned source commit: `c4ee360aac7afd2964785abac379a290d0c9847a` (SDK main branch).

when the SDK reaches stable release, switch to standard npm registry versions and remove the vendored tarballs.

## migration checklist

if upgrading from v1 or switching from vendored to registry:

- [ ] replace `@modelcontextprotocol/sdk` with the new split packages
- [ ] update all registration calls (`tool()` to `registerTool()`, etc.)
- [ ] replace `SSEServerTransport` with `NodeStreamableHTTPServerTransport`
- [ ] remove any server-side auth SDK usage; handle auth in middleware
- [ ] update Zod imports from `zod` to `zod/v4` if using Zod v4
- [ ] update `package.json` dependency versions from `file:` paths to registry versions
- [ ] delete `vendor/mcp-sdk-v2/` after switching to registry

## official references

- SDK repository: https://github.com/modelcontextprotocol/typescript-sdk
- migration guide: https://github.com/modelcontextprotocol/typescript-sdk/blob/main/docs/migration.md
- server guide: https://github.com/modelcontextprotocol/typescript-sdk/blob/main/docs/server.md

## next steps

- [validation](05-validation.md) -- verify the server and generated projects work correctly
