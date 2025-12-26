# /test - Run Tests

Run tests for MCP Inspector.

## Instructions

Run all tests:

```bash
npm test
```

This runs:

1. Prettier check (formatting)
2. Client tests (Vitest)

## Individual Test Commands

```bash
# Prettier check only
npm run prettier-check

# Client tests only
cd client && npm test

# CLI tests
npm run test-cli

# E2E tests (Playwright)
npm run test:e2e

# Lint check
npm run lint
```

## Test Structure

- `client/src/**/*.test.ts` - Client unit tests
- `cli/src/**/*.test.ts` - CLI tests
- `client/e2e/` - E2E tests with Playwright

## Fixing Issues

If Prettier check fails:

```bash
npm run prettier-fix
```

If lint fails:

```bash
cd client && npm run lint -- --fix
```
