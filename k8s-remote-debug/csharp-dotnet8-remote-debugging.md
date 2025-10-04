# Remote Debugging C# .NET 8 Web APIs in Kubernetes with VS Code

Debugging applications running in Kubernetes can be challenging, but with the right setup, you can debug C# .NET 8 Web APIs running in Kubernetes pods directly from your local VS Code instance. This post demonstrates remote debugging using vsdbg (Visual Studio Debugger) with kubectl exec transport.

## Overview

This example from the [k8s-vscode-remote-debug](https://github.com/nathanfox/k8s-vscode-remote-debug) repository showcases:
- Building debug-enabled Docker images for .NET 8
- Deploying to Kubernetes with debug configuration
- Attaching VS Code debugger to remote pods via kubectl exec
- Setting breakpoints and stepping through code running in Kubernetes

**What Works:**
- ✅ Full breakpoint support in all C# files
- ✅ Variable inspection (locals, parameters, closures)
- ✅ Call stack navigation
- ✅ Conditional breakpoints
- ✅ Expression evaluation in Debug Console
- ✅ Watch expressions
- ✅ Exception breakpoints

**Technology Stack:**
- **Language:** C#
- **Framework:** ASP.NET Core 8.0
- **Debugger:** vsdbg (Visual Studio Debugger)
- **Debug Method:** kubectl exec with pipeTransport
- **Container:** Multi-stage Docker build with debug mode

## How It Works

The debugging setup uses a unique approach compared to other languages:

1. **Docker Build:** The image includes vsdbg installed from `aka.ms/getvsdbgsh`
2. **VS Code Connection:** Uses kubectl exec with stdin/stdout piping (pipeTransport)
3. **No Port-Forwarding:** Direct connection to vsdbg in the pod via kubectl
4. **Process Attachment:** Attaches to the running .NET process (PID 1)

This approach differs from port-forwarding methods used by Node.js, Python, and Go, which can experience timeout issues on idle connections.

## Key Configuration

### Dockerfile (Debug Mode)

The debug build installs vsdbg for remote debugging:

```dockerfile
# Install vsdbg for remote debugging
RUN apt-get update && apt-get install -y curl unzip \
    && curl -sSL https://aka.ms/getvsdbgsh | /bin/sh /dev/stdin -v latest -l /vsdbg \
    && rm -rf /var/lib/apt/lists/*
```

See the [complete Dockerfile](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/csharp-dotnet8-webapi/Dockerfile) for the full multi-stage build configuration.

### VS Code launch.json

The debugger configuration uses kubectl exec with pipeTransport:

```json
{
  "name": "Attach to Remote Pod",
  "type": "coreclr",
  "request": "attach",
  "processId": "1",
  "pipeTransport": {
    "pipeCwd": "${workspaceRoot}",
    "pipeProgram": "kubectl",
    "pipeArgs": ["exec", "-i", "-n", "${input:namespace}", "${input:podName}", "--"],
    "debuggerPath": "/vsdbg/vsdbg",
    "quoteArgs": false
  },
  "sourceFileMap": {
    "/src/CSharpWebApi": "${workspaceFolder}/src/CSharpWebApi"
  }
}
```

The configuration prompts for namespace (defaulting to `${env:NAMESPACE}`) and pod name when you attach the debugger. See the [complete launch.json](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/csharp-dotnet8-webapi/.vscode/launch.json) with input definitions.

## Quick Start

```bash
# Set your developer namespace and registry
export NAMESPACE=dev-yourname
export REGISTRY=your-registry.azurecr.io  # Or docker.io/username, gcr.io/project, etc.

# Clone the repository
git clone https://github.com/nathanfox/k8s-vscode-remote-debug.git
cd k8s-vscode-remote-debug/examples/csharp-dotnet8-webapi

# Build, push, and deploy
./manage.sh build
./manage.sh push
./manage.sh deploy

# Verify pod is ready
./manage.sh debug

# Open in VS Code and press F5 to attach debugger
code .
```

## Example Debugging Session

1. **Set a breakpoint** in `Program.cs` at line 36 (inside the `/debug-test` handler)
2. **Attach debugger** by pressing F5 in VS Code
3. **Enter namespace and pod name** when prompted (use `kubectl get pods -n $NAMESPACE -l app=csharp-webapi` to find the pod)
4. **Port-forward the application** (in a separate terminal):
   ```bash
   ./manage.sh port-forward
   ```
5. **Trigger the endpoint:**
   ```bash
   curl http://localhost:8080/debug-test?count=3
   ```
6. **Breakpoint hits** - execution pauses at line 36
7. **Inspect variables:**
   - Hover over `itemCount` to see the value (3)
   - Check Variables panel to see `items`, `i`, `itemCount`
8. **Step through code:**
   - F10 (step over) to execute current line
   - Watch the `items` list grow as you step through the loop
9. **Continue execution** - F5 to resume

## VS Code Extensions

**Required for debugging:**
- **C#** (ms-dotnettools.csharp) - C# language support and debugger
- **C# Dev Kit** (ms-dotnettools.csdevkit) - Enhanced C# development experience

**Recommended (helpful but not required):**
- **Kubernetes** (ms-kubernetes-tools.vscode-kubernetes-tools) - K8s cluster management

## Important Troubleshooting Notes

### Liveness Probes
When debugging with breakpoints, your application stops responding to HTTP requests. If you have a liveness probe configured, Kubernetes will kill the pod after the timeout, causing exit code 137.

**Solution:** The example disables liveness probes for debug deployments:
```yaml
# Liveness probe disabled for debugging
# livenessProbe:
#   httpGet:
#     path: /health
#     port: 8080
```

The readiness probe still runs, marking the pod "Not Ready" when paused, preventing traffic from being routed to it.

### Debugger Won't Connect

If VS Code shows "Cannot connect to runtime process":

1. Verify NAMESPACE env var is set: `echo $NAMESPACE`
2. Check pod is running: `./manage.sh status`
3. Verify vsdbg is installed: `./manage.sh shell` then `ls -la /vsdbg/vsdbg`
4. Check kubectl connectivity: `kubectl exec -n $NAMESPACE -l app=csharp-webapi -- echo "Connected"`

## Example-Driven Development with AI Agents

This repository is designed to work seamlessly with AI coding assistants like [Claude Code](https://claude.ai/code). The complete, working examples allow AI agents to help you implement remote debugging for your own applications.

For more on this development pattern, see [Example-Driven Development Using AI Agent Claude Code](https://www.nathanfox.net/p/example-driven-development-using-ai-agent-claude-code).

**Example AI prompt:**
> "Using the k8s-vscode-remote-debug repository as a reference, add remote debugging support to my C# .NET 8 application. I want to debug it running in Kubernetes from local VS Code."

The AI can analyze your application and generate the appropriate Dockerfile, VS Code configuration, and Kubernetes manifests based on the working example.

## Next Steps

For complete details including:
- Full troubleshooting guide
- Integration with nginx-dev-gateway
- Additional debugging capabilities

See the [complete README](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/csharp-dotnet8-webapi/README.md) in the repository.

The repository includes examples for 8 languages/frameworks:
- C# .NET 8 Web API
- F# Giraffe
- Node.js Express
- Python FastAPI
- Go Gin
- Java Spring Boot
- Rust Actix (tracing-based)
- Elixir Phoenix (Remote-Kubernetes)

Each example provides a complete, working reference for implementing remote debugging in your own applications.
