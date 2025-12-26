# /dev - Start Development Server

Start the MCP Inspector development server.

## Instructions

1. Ensure dependencies are installed:

   ```bash
   test -d node_modules || npm install
   ```

2. Start the development server:
   ```bash
   npm run dev
   ```

This will start:

- Client (Vite dev server) on port 6274
- Server (Express backend)
- Auto-opens browser at http://localhost:6274

## Notes

- On Windows, use `npm run dev:windows` instead
- The dev server supports hot module replacement (HMR)
- Press Ctrl+C to stop the server

## For SDK Development

If you're developing against a local SDK:

```bash
npm run dev:sdk
```
