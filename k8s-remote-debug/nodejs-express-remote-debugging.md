# Remote Debugging Node.js Express Applications in Kubernetes with VS Code

Debugging Node.js applications running in Kubernetes is straightforward with the built-in Inspector Protocol. This post demonstrates remote debugging of Node.js Express applications using port-forwarding and VS Code, allowing you to debug async code, inspect promises, and step through your application running in Kubernetes.

## Overview

This example from the [k8s-vscode-remote-debug](https://github.com/nathanfox/k8s-vscode-remote-debug) repository showcases:
- Building Docker images with Node.js Inspector Protocol enabled
- Deploying to Kubernetes with debug port exposed
- Port-forwarding debug port (9229) from local machine to remote pod
- Attaching VS Code debugger using Node.js Inspector Protocol
- Debugging async/await code and promises

**What Works:**
- ✅ Breakpoints in JavaScript files
- ✅ Variable inspection (primitives, objects, arrays, closures)
- ✅ Call stack navigation (including async call stacks)
- ✅ Conditional breakpoints
- ✅ Expression evaluation in Debug Console
- ✅ Watch expressions
- ✅ Exception breakpoints
- ✅ Async/await debugging
- ✅ Promise inspection

**Technology Stack:**
- **Language:** Node.js (v22)
- **Framework:** Express
- **Debugger:** Node.js Inspector Protocol
- **Debug Method:** Port-forward to debug port 9229

## How It Works

The debugging setup uses Node.js's built-in Inspector Protocol:

1. **Docker Build:** Run Node.js with `--inspect=0.0.0.0:9229` flag
2. **Inspector Protocol:** Node.js built-in debug protocol over TCP
3. **Port-Forwarding:** VS Code connects via localhost:9229
4. **Debug Server:** Node.js listens on port 9229 for debugger connections

This approach uses standard TCP port-forwarding to connect your local VS Code to the remote Node.js debug server.

## Key Configuration

### Dockerfile (Debug Mode)

The Dockerfile runs Node.js with the `--inspect` flag:

```dockerfile
CMD ["node", "--inspect=0.0.0.0:9229", "src/index.js"]
```

This enables the Inspector Protocol on all interfaces (0.0.0.0) on port 9229. See the [complete Dockerfile](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/nodejs-express/Dockerfile) for the full configuration.

### VS Code launch.json

The debugger configuration attaches to the Inspector Protocol:

```json
{
  "name": "Attach to Remote Pod",
  "type": "node",
  "request": "attach",
  "address": "localhost",
  "port": 9229,
  "localRoot": "${workspaceFolder}/src",
  "remoteRoot": "/app/src",
  "skipFiles": ["<node_internals>/**"],
  "restart": true
}
```

Key settings:
- `address: "localhost"` - Connect to local port-forward
- `port: 9229` - Node.js Inspector Protocol default port
- `localRoot/remoteRoot` - Map local source files to container paths
- `restart: true` - Auto-reconnect if container restarts

See the [complete launch.json](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/nodejs-express/.vscode/launch.json) for all options.

## Quick Start

```bash
# Set your developer namespace and registry
export NAMESPACE=dev-yourname
export REGISTRY=your-registry.azurecr.io  # Or docker.io/username, gcr.io/project, etc.

# Clone the repository
git clone https://github.com/nathanfox/k8s-vscode-remote-debug.git
cd k8s-vscode-remote-debug/examples/nodejs-express

# Build, push, and deploy
./manage.sh build
./manage.sh push
./manage.sh deploy

# Verify pod is ready and port-forward debug port
./manage.sh debug

# Open in VS Code and press F5 to attach debugger
code .
```

## Example Debugging Session

1. **Set a breakpoint** in `src/index.js` at line 15 (inside the `/debug-test` handler)
2. **Port-forward debug port** (in a separate terminal):
   ```bash
   ./manage.sh debug
   ```
3. **Attach debugger** by pressing F5 in VS Code
4. **Port-forward the application** (in another terminal):
   ```bash
   ./manage.sh port-forward
   ```
5. **Trigger the endpoint:**
   ```bash
   curl http://localhost:8080/debug-test?count=3
   ```
6. **Breakpoint hits** - execution pauses at line 15
7. **Inspect variables:**
   - Hover over `count` to see the value (3)
   - Check Variables panel to see `items`, `i`, `req`, `res`
   - Inspect async context and promises
8. **Step through code:**
   - F10 (step over) to execute current line
   - F11 (step into) to step into async functions
   - Watch the `items` array grow as you step through the loop
9. **Continue execution** - F5 to resume

## Debugging Async/Await Code

Node.js debugging excels at async operations:

**Async Call Stacks:**
- Full async call stack visible in VS Code
- See the entire promise chain
- Understand async flow and timing

**Promise Inspection:**
- View promise states (pending, fulfilled, rejected)
- Inspect resolved/rejected values
- Catch unhandled promise rejections

**Example async handler:**
```javascript
app.get('/debug-test', async (req, res) => {
  const count = parseInt(req.query.count) || 3;
  const items = [];

  for (let i = 0; i < count; i++) {
    items.push(`Item ${i + 1}`);
    await new Promise(resolve => setTimeout(resolve, 10));  // Breakpoint here
  }

  res.json({ count, items, timestamp: new Date().toISOString() });
});
```

Set a breakpoint inside the loop to watch async delays execute step-by-step.

## VS Code Extensions

**Recommended (helpful but not required):**
- **ESLint** (dbaeumer.vscode-eslint) - JavaScript linting
- **Kubernetes** (ms-kubernetes-tools.vscode-kubernetes-tools) - K8s cluster management

The Node.js debugger is built into VS Code - no additional extensions required for debugging!

## Troubleshooting

### Debugger Won't Connect

If VS Code shows "Cannot connect to runtime process":

1. **Verify port-forward is running:**
   ```bash
   ps aux | grep "port-forward.*9229"
   ```

2. **Check pod is running:**
   ```bash
   ./manage.sh status
   ```

3. **Verify Node.js is listening on 9229:**
   ```bash
   ./manage.sh logs | grep "Debugger listening"
   ```

4. **Restart port-forward:**
   ```bash
   pkill -f "port-forward.*9229"
   ./manage.sh debug
   ```

### Port-Forward Connection Drops

**Symptom:** Debugger disconnects randomly

**Solutions:**
- kubectl port-forward can timeout on idle connections
- Restart port-forward when needed: `./manage.sh debug`
- The `restart: true` setting in launch.json helps auto-reconnect

### Breakpoints Not Hitting

1. **Verify source path mapping:**
   - `localRoot: "${workspaceFolder}/src"`
   - `remoteRoot: "/app/src"`

2. **Check request is reaching the endpoint:**
   ```bash
   ./manage.sh logs --follow
   ```

## Example-Driven Development with AI Agents

This repository demonstrates Example-Driven Development, designed to work with AI coding assistants like [Claude Code](https://claude.ai/code).

For more on this pattern, see [Example-Driven Development Using AI Agent Claude Code](https://www.nathanfox.net/p/example-driven-development-using-ai-agent-claude-code).

**Example AI prompt:**
> "Using the k8s-vscode-remote-debug repository's Node.js Express example, add remote debugging support to my Express application running in Kubernetes."

The AI can generate the appropriate Dockerfile, launch.json, and deployment configuration based on the working example.

## Next Steps

For complete details including:
- Full troubleshooting guide
- Hot reload with nodemon
- Memory profiling with Inspector Protocol
- Performance considerations

See the [complete README](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/nodejs-express/README.md) in the repository.

The repository includes examples for 8 languages/frameworks, each demonstrating the unique aspects of debugging that language in Kubernetes.
