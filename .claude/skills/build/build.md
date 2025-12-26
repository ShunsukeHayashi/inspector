# /build - Build Project

Build all MCP Inspector packages (client, server, cli).

## Instructions

Run the full build:

```bash
npm run build
```

This executes in order:

1. `npm run build-server` - Build Express server
2. `npm run build-client` - Build React client with Vite
3. `npm run build-cli` - Build CLI tool

## Individual Builds

To build specific packages:

```bash
# Server only
npm run build-server

# Client only
npm run build-client

# CLI only
npm run build-cli
```

## Output

Build artifacts are created in:

- `server/build/` - Server JavaScript files
- `client/dist/` - Client bundle
- `cli/build/` - CLI executable

## Verification

After building, verify:

```bash
ls -la server/build client/dist cli/build
```
