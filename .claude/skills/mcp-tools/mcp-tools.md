# MCP Tools - Add Custom Tools

Guide for adding custom tools to the VPS MCP server.

## Tool Structure

```javascript
mcpServer.tool(
  "tool_name", // Unique identifier
  "Tool description", // Human-readable description
  {
    // Input schema (JSON Schema)
    param1: { type: "string", description: "Parameter 1" },
    param2: { type: "number", description: "Parameter 2" },
  },
  async (args) => {
    // Handler function
    // Your logic here
    return {
      content: [
        {
          type: "text",
          text: "Result",
        },
      ],
    };
  },
);
```

## Example Tools

### File Reader

```javascript
mcpServer.tool(
  "read_file",
  "Read a file from the server",
  {
    path: { type: "string", description: "File path to read" },
  },
  async (args) => {
    const fs = await import("fs/promises");
    try {
      const content = await fs.readFile(args.path, "utf-8");
      return { content: [{ type: "text", text: content }] };
    } catch (e) {
      return { content: [{ type: "text", text: `Error: ${e.message}` }] };
    }
  },
);
```

### Command Executor

```javascript
mcpServer.tool(
  "exec_command",
  "Execute a shell command",
  {
    command: { type: "string", description: "Command to execute" },
  },
  async (args) => {
    const { exec } = await import("child_process");
    const { promisify } = await import("util");
    const execAsync = promisify(exec);

    try {
      const { stdout, stderr } = await execAsync(args.command, {
        timeout: 30000,
      });
      return { content: [{ type: "text", text: stdout || stderr }] };
    } catch (e) {
      return { content: [{ type: "text", text: `Error: ${e.message}` }] };
    }
  },
);
```

### Docker Status

```javascript
mcpServer.tool("docker_ps", "List Docker containers", {}, async () => {
  const { exec } = await import("child_process");
  const { promisify } = await import("util");
  const execAsync = promisify(exec);

  try {
    const { stdout } = await execAsync(
      "docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'",
    );
    return { content: [{ type: "text", text: stdout }] };
  } catch (e) {
    return { content: [{ type: "text", text: `Error: ${e.message}` }] };
  }
});
```

### System Metrics

```javascript
mcpServer.tool(
  "system_metrics",
  "Get detailed system metrics",
  {},
  async () => {
    const os = await import("os");
    const { exec } = await import("child_process");
    const { promisify } = await import("util");
    const execAsync = promisify(exec);

    const cpuUsage = os.loadavg();
    const { stdout: diskInfo } = await execAsync("df -h / | tail -1");

    return {
      content: [
        {
          type: "text",
          text: JSON.stringify(
            {
              cpu: {
                cores: os.cpus().length,
                load_1m: cpuUsage[0].toFixed(2),
                load_5m: cpuUsage[1].toFixed(2),
                load_15m: cpuUsage[2].toFixed(2),
              },
              memory: {
                total_gb: (os.totalmem() / 1024 ** 3).toFixed(2),
                used_gb: ((os.totalmem() - os.freemem()) / 1024 ** 3).toFixed(
                  2,
                ),
                percent:
                  ((1 - os.freemem() / os.totalmem()) * 100).toFixed(1) + "%",
              },
              disk: diskInfo.trim(),
            },
            null,
            2,
          ),
        },
      ],
    };
  },
);
```

### HTTP Request

```javascript
mcpServer.tool(
  "http_get",
  "Make an HTTP GET request",
  {
    url: { type: "string", description: "URL to fetch" },
  },
  async (args) => {
    try {
      const res = await fetch(args.url);
      const text = await res.text();
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify(
              {
                status: res.status,
                headers: Object.fromEntries(res.headers),
                body: text.substring(0, 1000),
              },
              null,
              2,
            ),
          },
        ],
      };
    } catch (e) {
      return { content: [{ type: "text", text: `Error: ${e.message}` }] };
    }
  },
);
```

## Deploy Updated Tools

After adding tools to server.js:

```bash
# Copy to VPS
scp /path/to/server.js root@85.131.245.74:/opt/mcp-server/

# Restart server
ssh root@85.131.245.74 'pm2 restart mcp-server'

# Verify
curl http://85.131.245.74:3000/health
```

## Test Tools in Inspector

1. Open MCP Inspector
2. Connect to `http://85.131.245.74:3000/mcp`
3. Go to **Tools** tab
4. Select your new tool
5. Fill in parameters
6. Click **Execute**
