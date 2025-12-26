# VPS Deploy - Automated Deployment

Deploy applications to XServer VPS (85.131.245.74).

## Prerequisites

Ensure SSH key is configured:

```bash
ssh-copy-id root@85.131.245.74
```

## Deploy MCP Server

### One-liner Deploy

```bash
ssh root@85.131.245.74 'bash -s' << 'DEPLOY'
set -e

# Install Node.js if not present
if ! command -v node &> /dev/null; then
    curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
    apt install -y nodejs
fi

# Install PM2 if not present
if ! command -v pm2 &> /dev/null; then
    npm install -g pm2
fi

# Create MCP server directory
mkdir -p /opt/mcp-server
cd /opt/mcp-server

# Create package.json
cat > package.json << 'EOF'
{
  "name": "lunch-dev-mcp",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.24.0",
    "express": "^4.21.0"
  }
}
EOF

# Create server.js
cat > server.js << 'EOF'
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import express from "express";
import os from "os";
import { exec } from "child_process";
import { promisify } from "util";

const execAsync = promisify(exec);
const app = express();
app.use(express.json());

const mcpServer = new McpServer({
  name: "lunch-dev-mcp",
  version: "1.0.0",
});

// Tool: Hello
mcpServer.tool("hello", "Say hello to someone", {
  name: { type: "string", description: "Name to greet" }
}, async (args) => {
  return { content: [{ type: "text", text: `Hello, ${args.name}!` }] };
});

// Tool: Server Status
mcpServer.tool("server_status", "Get VPS server status", {}, async () => {
  return {
    content: [{
      type: "text",
      text: JSON.stringify({
        hostname: os.hostname(),
        uptime: `${Math.floor(os.uptime() / 3600)}h ${Math.floor((os.uptime() % 3600) / 60)}m`,
        memory: {
          total: `${(os.totalmem() / 1024 / 1024 / 1024).toFixed(2)} GB`,
          free: `${(os.freemem() / 1024 / 1024 / 1024).toFixed(2)} GB`,
          used: `${((os.totalmem() - os.freemem()) / 1024 / 1024 / 1024).toFixed(2)} GB`,
        },
        cpus: os.cpus().length,
        platform: os.platform(),
        arch: os.arch(),
      }, null, 2),
    }],
  };
});

// Tool: Disk Usage
mcpServer.tool("disk_usage", "Get disk usage", {}, async () => {
  try {
    const { stdout } = await execAsync("df -h /");
    return { content: [{ type: "text", text: stdout }] };
  } catch (e) {
    return { content: [{ type: "text", text: `Error: ${e.message}` }] };
  }
});

// Tool: Process List
mcpServer.tool("process_list", "List running processes", {
  filter: { type: "string", description: "Filter by name (optional)" }
}, async (args) => {
  try {
    const cmd = args.filter
      ? `ps aux | grep -i "${args.filter}" | head -20`
      : "ps aux --sort=-%mem | head -15";
    const { stdout } = await execAsync(cmd);
    return { content: [{ type: "text", text: stdout }] };
  } catch (e) {
    return { content: [{ type: "text", text: `Error: ${e.message}` }] };
  }
});

// Tool: Network Info
mcpServer.tool("network_info", "Get network information", {}, async () => {
  const interfaces = os.networkInterfaces();
  const result = {};
  for (const [name, addrs] of Object.entries(interfaces)) {
    result[name] = addrs.filter(a => a.family === "IPv4").map(a => a.address);
  }
  return { content: [{ type: "text", text: JSON.stringify(result, null, 2) }] };
});

// Streamable HTTP transport
const transport = new StreamableHTTPServerTransport({ endpoint: "/mcp" });

app.all("/mcp", async (req, res) => {
  await transport.handleRequest(req, res, mcpServer);
});

app.get("/health", (req, res) => {
  res.json({ status: "ok", server: "lunch-dev-mcp", uptime: os.uptime() });
});

app.get("/", (req, res) => {
  res.json({
    name: "lunch-dev-mcp",
    version: "1.0.0",
    endpoints: {
      mcp: "/mcp",
      health: "/health"
    }
  });
});

const PORT = 3000;
app.listen(PORT, "0.0.0.0", () => {
  console.log(`MCP Server: http://85.131.245.74:${PORT}`);
  console.log(`Streamable HTTP: http://85.131.245.74:${PORT}/mcp`);
});
EOF

# Install dependencies
npm install

# Configure firewall
ufw allow 3000/tcp 2>/dev/null || true

# Start/restart with PM2
pm2 delete mcp-server 2>/dev/null || true
pm2 start server.js --name mcp-server
pm2 save

echo "✅ MCP Server deployed successfully!"
echo "📡 Endpoint: http://85.131.245.74:3000/mcp"
DEPLOY
```

## Verify Deployment

```bash
# Check health endpoint
curl http://85.131.245.74:3000/health

# Check PM2 status
ssh root@85.131.245.74 'pm2 status'
```

## Update Server

After modifying server.js locally:

```bash
scp server.js root@85.131.245.74:/opt/mcp-server/
ssh root@85.131.245.74 'pm2 restart mcp-server'
```

## View Logs

```bash
ssh root@85.131.245.74 'pm2 logs mcp-server --lines 50'
```
