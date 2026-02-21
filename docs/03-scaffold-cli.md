# scaffold CLI

generate new MCP stateful HTTP projects from the included templates.

## init command

```bash
npm run create -- init <project-name> [options]
```

or from built output:

```bash
node dist/cli/index.js init <project-name> [options]
```

or as a global/linked binary:

```bash
mcp-stateful-starter init <project-name> [options]
```

### options

| flag                    | default     | description                                      |
|-------------------------|-------------|--------------------------------------------------|
| `--force`               | `false`     | overwrite files in an existing non-empty directory |
| `--dir <path>`          | `cwd`       | parent directory for the generated project       |
| `--sdk vendored\|registry` | `vendored`  | SDK dependency mode (see below)                  |

### examples

```bash
# basic usage
npm run create -- init my-mcp-app

# place in a specific directory
npm run create -- init my-mcp-app --dir ./playground

# overwrite existing files
npm run create -- init my-mcp-app --force

# use registry packages instead of vendored tarballs
npm run create -- init my-mcp-app --sdk registry
```

## SDK modes

### vendored (default)

copies the tarballs from `vendor/mcp-sdk-v2/` into the generated project and sets `package.json` dependencies to `file:vendor/mcp-sdk-v2/*.tgz`. this ensures reproducible installs regardless of npm registry state.

### registry

sets `package.json` dependencies to `2.0.0-alpha.0` version strings, resolving directly from the npm registry. use this when the v2 packages are published and you want standard npm resolution.

## generated structure

```
<project-name>/
  .env.example
  .gitignore
  package.json
  tsconfig.json
  README.md
  src/
    index.ts          # single-file server with all MCP wiring
  vendor/             # only present in vendored mode
    mcp-sdk-v2/
      *.tgz
```

the generated `src/index.ts` is a self-contained server derived from the `templates/http-stateful/src/index.ts.tmpl` template. template variables (`__PROJECT_NAME__`, `__SDK_SERVER_DEP__`, `__SDK_NODE_DEP__`) are replaced during generation.

## validation checklist

after generating a project:

```bash
cd <project-name>
npm install
npm run dev
# verify http://127.0.0.1:1453/health returns { ok: true }
```

- [ ] `npm install` completes without errors
- [ ] `npm run dev` starts the server on port 1453
- [ ] `/health` returns `{ ok: true }`
- [ ] `POST /mcp` with an initialize request returns an `Mcp-Session-Id` header

## next steps

- [SDK v2 notes](04-sdk-v2-notes.md) -- understand the v2 package split and vendoring strategy
- [validation](05-validation.md) -- full verification and release checklist
