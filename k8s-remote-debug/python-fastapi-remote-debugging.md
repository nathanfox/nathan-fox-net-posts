# Remote Debugging Python FastAPI Applications in Kubernetes with VS Code

Debugging Python applications running in Kubernetes is possible with debugpy, Microsoft's debug adapter for Python. This post demonstrates remote debugging of Python FastAPI applications using debugpy and VS Code, allowing you to debug async code, inspect coroutines, and step through your application running in Kubernetes.

## Overview

This example from the [k8s-vscode-remote-debug](https://github.com/nathanfox/k8s-vscode-remote-debug) repository showcases:
- Building Docker images with debugpy enabled
- Deploying to Kubernetes with debug configuration
- Port-forwarding debug port (5678) from local machine to remote pod
- Attaching VS Code debugger using debugpy
- Debugging async/await code and coroutines

**What Works:**
- ✅ Breakpoints in Python files
- ✅ Variable inspection (primitives, objects, lists, dicts)
- ✅ Call stack navigation (including async call stacks)
- ✅ Conditional breakpoints
- ✅ Expression evaluation in Debug Console
- ✅ Watch expressions
- ✅ Exception breakpoints
- ✅ Async/await debugging
- ✅ Coroutine inspection

**Technology Stack:**
- **Language:** Python 3.12
- **Framework:** FastAPI
- **Debugger:** debugpy
- **Debug Method:** Port-forward to debug port 5678

## How It Works

The debugging setup uses debugpy, Python's debug adapter protocol implementation:

1. **Code Integration:** Import debugpy and call `debugpy.listen()` in application code
2. **Debug Server:** debugpy listens on port 5678 for connections
3. **Port-Forwarding:** VS Code connects via localhost:5678
4. **Debug Adapter Protocol:** debugpy implements DAP for VS Code

Unlike some languages where debugging is enabled via command-line flags, Python requires explicit debugpy integration in your application code.

## Key Configuration

### Application Code (main.py)

Debugpy must be initialized in your Python application:

```python
import debugpy
import os

DEBUG_PORT = int(os.getenv("DEBUG_PORT", "5678"))
ENABLE_DEBUGPY = os.getenv("ENABLE_DEBUGPY", "true").lower() == "true"

if ENABLE_DEBUGPY:
    debugpy.listen(("0.0.0.0", DEBUG_PORT))
    print(f"debugpy listening on 0.0.0.0:{DEBUG_PORT}")
```

**Key points:**
- Use `ENABLE_DEBUGPY` environment variable to toggle debugging
- Listen on `0.0.0.0` to accept remote connections
- Port 5678 is debugpy's default port

See the [complete main.py](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/python-fastapi/src/main.py) for the full application code.

### Dockerfile (Debug Mode)

The Dockerfile installs debugpy as a dependency:

```dockerfile
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
```

With debugpy listed in [requirements.txt](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/python-fastapi/requirements.txt):

```txt
fastapi==0.115.0
uvicorn[standard]==0.32.0
debugpy==1.8.7
```

See the [complete Dockerfile](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/python-fastapi/Dockerfile) for the full build.

### Kubernetes Deployment

Enable debugging via environment variable:

```yaml
env:
- name: ENABLE_DEBUGPY
  value: "true"
- name: DEBUG_PORT
  value: "5678"
```

See the [complete deployment.yaml](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/python-fastapi/k8s/deployment.yaml) for full configuration.

### VS Code launch.json

The debugger configuration attaches to debugpy:

```json
{
  "name": "Attach to Remote Pod",
  "type": "debugpy",
  "request": "attach",
  "connect": {
    "host": "localhost",
    "port": 5678
  },
  "pathMappings": [
    {
      "localRoot": "${workspaceFolder}/src",
      "remoteRoot": "/app/src"
    }
  ],
  "justMyCode": false
}
```

Key settings:
- `type: "debugpy"` - Use debugpy debug adapter
- `port: 5678` - debugpy default port
- `pathMappings` - Map local source to container paths
- `justMyCode: false` - Allow stepping into library code

See the [complete launch.json](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/python-fastapi/.vscode/launch.json) for all options.

## Quick Start

```bash
# Set your developer namespace and registry
export NAMESPACE=dev-yourname
export REGISTRY=your-registry.azurecr.io  # Or docker.io/username, gcr.io/project, etc.

# Clone the repository
git clone https://github.com/nathanfox/k8s-vscode-remote-debug.git
cd k8s-vscode-remote-debug/examples/python-fastapi

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

1. **Set a breakpoint** in `src/main.py` at line 28 (inside the `/debug-test` handler)
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
6. **Breakpoint hits** - execution pauses at line 28
7. **Inspect variables:**
   - Hover over `count` to see the value (3)
   - Check Variables panel to see `items`, `i`
   - Inspect async context
8. **Step through code:**
   - F10 (step over) to execute current line
   - F11 (step into) to step into async functions
   - Watch the `items` list grow as you step through the loop
9. **Continue execution** - F5 to resume

## Debugging Async/Await Code

FastAPI is async-first, making async debugging essential:

### Async Call Stacks

- Full async call stack visible in VS Code
- See the entire coroutine chain
- Understand async flow and awaits

### Coroutine Inspection

- View coroutine states (running, suspended)
- Inspect awaited values
- Debug async context managers

### Example async handler:

```python
@app.get("/debug-test")
async def debug_test(count: int = Query(default=3, ge=1, le=20)):
    items = []

    for i in range(count):
        items.append(f"Item {i + 1}")
        await asyncio.sleep(0.01)  # Breakpoint here - inspect await

    return {
        "count": count,
        "items": items,
        "timestamp": datetime.utcnow().isoformat()
    }
```

Set breakpoints on `await` statements to inspect async operations.

## FastAPI Framework Debugging

**FastAPI** is a modern async Python web framework:

- Built on Starlette and Pydantic
- Async-first design (native async/await)
- Automatic request validation via type hints
- Query parameter parsing via function parameters

Debugging features:
- Inspect request parameters (`count` in example)
- View Pydantic model validation
- Debug dependency injection
- Inspect response serialization

## VS Code Extensions

**Required for debugging:**
- **Python** (ms-python.python) - Python language support and debugging
- **Debugpy** (ms-python.debugpy) - Python debug adapter (often included with Python extension)

**Recommended (helpful but not required):**
- **Kubernetes** (ms-kubernetes-tools.vscode-kubernetes-tools) - K8s cluster management

The Python extension typically includes debugpy support, but installing both ensures compatibility.

## Troubleshooting

### Debugger Won't Connect

If VS Code shows "Cannot connect to runtime process":

1. **Verify port-forward is running:**
   ```bash
   ps aux | grep "port-forward.*5678"
   ```

2. **Check pod is running:**
   ```bash
   ./manage.sh status
   ```

3. **Verify debugpy is listening:**
   ```bash
   ./manage.sh logs | grep "debugpy listening"
   ```

4. **Restart port-forward:**
   ```bash
   pkill -f "port-forward.*5678"
   ./manage.sh debug
   ```

### Breakpoints Not Hitting

1. **Ensure source paths match:**
   - Check `pathMappings` in launch.json
   - `localRoot: "${workspaceFolder}/src"`
   - `remoteRoot: "/app/src"`

2. **Verify files are mounted correctly:**
   ```bash
   ./manage.sh shell
   ls -la /app/src  # Should show main.py
   ```

3. **Check debugpy is attached:**
   - VS Code should show "Python: Attached" in status bar
   - Debug Console should show connection message

### Port-Forward Connection Drops

**Symptom:** Debugger disconnects randomly

**Solutions:**
- kubectl port-forward can timeout on idle connections
- Restart port-forward when needed: `./manage.sh debug`

## Enabling/Disabling Debug Mode

To toggle debugging without rebuilding:

**Enable debugging:**
```bash
kubectl set env deployment/python-fastapi ENABLE_DEBUGPY=true -n $NAMESPACE
```

**Disable debugging:**
```bash
kubectl set env deployment/python-fastapi ENABLE_DEBUGPY=false -n $NAMESPACE
```

This allows using the same Docker image in both debug and production environments.

## Example-Driven Development with AI Agents

This repository demonstrates Example-Driven Development, designed to work with AI coding assistants like [Claude Code](https://claude.ai/code).

For more on this pattern, see [Example-Driven Development Using AI Agent Claude Code](https://www.nathanfox.net/p/example-driven-development-using-ai-agent-claude-code).

**Example AI prompt:**
> "Using the k8s-vscode-remote-debug repository's Python FastAPI example, add remote debugging support to my FastAPI application running in Kubernetes."

The AI can generate the appropriate debugpy integration, Dockerfile configuration, and deployment manifests based on the working example.

## Next Steps

For complete details including:
- Full troubleshooting guide
- Hot reload with uvicorn --reload
- Performance considerations
- Production debugging best practices

See the [complete README](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/python-fastapi/README.md) in the repository.

The repository includes examples for 8 languages/frameworks, each demonstrating the unique aspects of debugging that language in Kubernetes.
