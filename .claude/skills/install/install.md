# /install - Install Dependencies

Install all dependencies for MCP Inspector.

## Instructions

Install all workspace dependencies:

```bash
npm install
```

This installs dependencies for:

- Root package
- `client/` workspace
- `server/` workspace
- `cli/` workspace

## Verification

After installation, verify:

```bash
ls node_modules client/node_modules server/node_modules cli/node_modules
```

## Troubleshooting

If installation fails:

1. **Clean install**:

   ```bash
   rm -rf node_modules package-lock.json
   npm install
   ```

2. **Node version check** (requires Node >= 22.7.5):

   ```bash
   node --version
   ```

3. **Full clean**:
   ```bash
   npm run clean
   ```

## Post-Install

The `prepare` script automatically runs after install:

- Sets up Husky (git hooks)
- Builds all packages
