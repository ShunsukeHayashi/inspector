# /ccg - AI Course Content Generator

Instructions for using the MCP Inspector with MCP servers.

## What is MCP Inspector?

MCP Inspector is a visual tool for testing and debugging MCP (Model Context Protocol) servers. It allows you to:

- Connect to any MCP server
- Browse available tools, resources, and prompts
- Execute tool calls interactively
- View request/response logs
- Test server behavior

## How to Use

### 1. Start the Inspector

```bash
# Development mode
npm run dev

# Or production mode
npm start
```

Opens at: http://localhost:6274

### 2. Connect to an MCP Server

**Option A: Command-based server**

```
Command: npx
Arguments: -y @modelcontextprotocol/server-github
Environment: GITHUB_TOKEN=your_token
```

**Option B: URL-based server (SSE)**

```
Transport: SSE
URL: http://localhost:8080/sse
```

### 3. Explore the Server

After connecting:

- **Tools Tab**: View and execute available tools
- **Resources Tab**: Browse server resources
- **Prompts Tab**: Test prompt templates
- **Logs Tab**: View request/response history

### 4. CLI Usage

Use the CLI for direct testing:

```bash
# Install globally
npm install -g @modelcontextprotocol/inspector

# Run inspector with a server
mcp-inspector npx -y @modelcontextprotocol/server-github
```

## Example: Testing GitHub MCP Server

1. Start inspector: `npm run dev`
2. Enter command: `npx -y @modelcontextprotocol/server-github`
3. Add env: `GITHUB_TOKEN=ghp_xxx`
4. Click "Connect"
5. Go to Tools tab
6. Try `search_repositories` with query `mcp`

## Troubleshooting

- **Connection fails**: Check server command and environment variables
- **Tools not showing**: Ensure server implements `tools/list`
- **Errors in logs**: Check Logs tab for detailed error messages
