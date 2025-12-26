# /clean - Cleanup

Clean up build artifacts and reinstall dependencies.

## Instructions

Run full cleanup:

```bash
npm run clean
```

This removes:

- `node_modules/` (root)
- `client/node_modules/`
- `cli/node_modules/`
- `build/`
- `client/dist/`
- `server/build/`
- `cli/build/`
- `package-lock.json`

Then automatically runs `npm install`.

## Manual Cleanup

For partial cleanup:

```bash
# Remove build artifacts only
rm -rf client/dist server/build cli/build

# Remove node_modules only
rm -rf node_modules client/node_modules server/node_modules cli/node_modules

# Remove lock file
rm -f package-lock.json
```

## When to Use

Use `/clean` when:

- Dependency issues occur
- Build artifacts are corrupted
- Switching between Node versions
- After major dependency updates
