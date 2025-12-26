# VPS MCP Server - Streamable HTTP

Deploy and manage an MCP server with Streamable HTTP transport on XServer VPS.

## VPS Information

- **Server**: lunch_dev
- **IP**: 85.131.245.74
- **OS**: Ubuntu 24.04
- **Specs**: 6 vCPU, 8GB RAM, 400GB NVMe

## Quick Deploy

### 1. SSH to VPS

```bash
ssh root@85.131.245.74
```

### 2. Install Dependencies

```bash
# Update system
apt update && apt upgrade -y

# Install Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# Install PM2 for process management
npm install -g pm2

# Install build tools
apt install -y build-essential git
```

### 3. Clone/Create MCP Server

```bash
mkdir -p /opt/mcp-server
cd /opt/mcp-server
npm init -y
npm install @modelcontextprotocol/sdk express
```

### 4. Create Server Code

Create `/opt/mcp-server/server.js`:

```javascript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import express from "express";

const app = express();
app.use(express.json());

// Create MCP server
const mcpServer = new McpServer({
  name: "lunch-dev-mcp",
  version: "1.0.0",
});

// Register tools
mcpServer.tool(
  "hello",
  "Say hello",
  { name: { type: "string" } },
  async (args) => {
    return { content: [{ type: "text", text: `Hello, ${args.name}!` }] };
  },
);

mcpServer.tool("server_status", "Get server status", {}, async () => {
  const os = await import("os");
  return {
    content: [
      {
        type: "text",
        text: JSON.stringify(
          {
            hostname: os.hostname(),
            uptime: os.uptime(),
            memory: {
              total: os.totalmem(),
              free: os.freemem(),
            },
            cpus: os.cpus().length,
          },
          null,
          2,
        ),
      },
    ],
  };
});

// Streamable HTTP endpoint
const transport = new StreamableHTTPServerTransport({ endpoint: "/mcp" });

app.all("/mcp", async (req, res) => {
  await transport.handleRequest(req, res, mcpServer);
});

// Health check
app.get("/health", (req, res) => {
  res.json({ status: "ok", server: "lunch-dev-mcp" });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, "0.0.0.0", () => {
  console.log(`MCP Server running on http://0.0.0.0:${PORT}`);
  console.log(`Streamable HTTP endpoint: http://85.131.245.74:${PORT}/mcp`);
});
```

### 5. Create package.json

```json
{
  "name": "lunch-dev-mcp",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node server.js",
    "dev": "node --watch server.js"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.24.0",
    "express": "^4.21.0"
  }
}
```

### 6. Start with PM2

```bash
cd /opt/mcp-server
pm2 start server.js --name mcp-server
pm2 save
pm2 startup
```

### 7. Configure Firewall

```bash
ufw allow 3000/tcp
ufw reload
```

## Connect from Inspector

1. Start MCP Inspector locally
2. Select Transport: **Streamable HTTP**
3. URL: `http://85.131.245.74:3000/mcp`
4. Click Connect

## Management Commands

```bash
# View logs
pm2 logs mcp-server

# Restart
pm2 restart mcp-server

# Stop
pm2 stop mcp-server

# Status
pm2 status
```

## Add Custom Tools

Edit `/opt/mcp-server/server.js` and add:

```javascript
mcpServer.tool(
  "my_tool",
  "Description",
  {
    param1: { type: "string", description: "Parameter 1" },
  },
  async (args) => {
    // Your logic here
    return { content: [{ type: "text", text: "Result" }] };
  },
);
```

Then restart: `pm2 restart mcp-server`
