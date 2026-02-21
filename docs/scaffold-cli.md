# Scaffold Creator CLI

This repo includes a project generator CLI:

- command name: `mcp-http-stateful-starter`
- source: `src/cli/index.ts`

## Usage

From repo scripts:

```bash
npm run create -- init my-mcp-app
```

From built output:

```bash
node dist/cli/index.js init my-mcp-app
```

Help:

```bash
node dist/cli/index.js --help
```

## Options

- `--force`
  Allow generating into an existing non-empty target directory.
- `--dir <path>`
  Base directory where the project folder will be created.
- `--sdk vendored|registry`
  Dependency strategy for SDK v2 packages.

## SDK Modes

### `vendored` (default)

Copies:

- `vendor/mcp-sdk-v2/modelcontextprotocol-server-2.0.0-alpha.0.tgz`
- `vendor/mcp-sdk-v2/modelcontextprotocol-node-2.0.0-alpha.0.tgz`

into the generated project and wires `package.json` to local tarball paths.

Use this mode when you need deterministic installs while v2 is pre-release.

### `registry`

Writes semver package deps (`2.0.0-alpha.0`) for `@modelcontextprotocol/server` and `@modelcontextprotocol/node`.

Use this only if your npm environment resolves these pre-release packages.

## Generated Structure

The generated project includes:

- `src/index.ts` (runnable stateful Streamable HTTP server)
- `package.json`, `tsconfig.json`
- `.env.example`, `.gitignore`
- `README.md`
- optional `vendor/mcp-sdk-v2/` folder (vendored mode)

## Validation Result

Most recent local validation in this repository:

1. CLI generated project successfully.
2. Generated project passed:
   - `npm install`
   - `npm run typecheck`
   - `npm run build`
3. Generated project server returned healthy response from `/health`.
