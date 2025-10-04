# Remote Debugging F# Giraffe Applications in Kubernetes with VS Code

Debugging functional F# applications running in Kubernetes is possible with the right setup. This post demonstrates remote debugging of F# Giraffe web applications using vsdbg and VS Code, allowing you to debug functional code, pipelines, and pattern matching directly in your Kubernetes environment.

## Overview

This example from the [k8s-vscode-remote-debug](https://github.com/nathanfox/k8s-vscode-remote-debug) repository showcases:
- Building debug-enabled Docker images for F# with Giraffe framework
- Deploying to Kubernetes with debug configuration
- Attaching VS Code debugger to remote pods via kubectl exec
- Debugging functional programming constructs (pipelines, pattern matching, closures)

**What Works:**
- ✅ Breakpoints in F# code
- ✅ Variable inspection (including function closures)
- ✅ Call stack navigation (including functional composition)
- ✅ Conditional breakpoints
- ✅ Expression evaluation in Debug Console
- ✅ Watch expressions
- ✅ Pipeline debugging (F# pipe operators)
- ✅ Pattern matching inspection

**Technology Stack:**
- **Language:** F#
- **Framework:** Giraffe on ASP.NET Core 8.0
- **Debugger:** vsdbg (Visual Studio Debugger)
- **Debug Method:** kubectl exec with pipeTransport

## How It Works

F# debugging works identically to C# since both run on the .NET runtime:

1. **Docker Build:** Image includes vsdbg from `aka.ms/getvsdbgsh`
2. **VS Code Connection:** Uses kubectl exec with pipeTransport
3. **No Port-Forwarding:** Direct connection via kubectl
4. **Process Attachment:** Attaches to the .NET process (PID 1)

## Key Configuration

### Dockerfile (Debug Mode)

The debug build installs vsdbg for remote debugging. See the [complete Dockerfile](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/fsharp-giraffe-dotnet8/Dockerfile) for the full multi-stage build configuration.

## Key Configuration

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
    "/src/FSharpGiraffe": "${workspaceFolder}/src/FSharpGiraffe"
  }
}
```

This configuration prompts for namespace and pod name, providing flexibility for multi-pod debugging scenarios. See the [complete launch.json](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/fsharp-giraffe-dotnet8/.vscode/launch.json) with input definitions.

## Quick Start

```bash
# Set your developer namespace and registry
export NAMESPACE=dev-yourname
export REGISTRY=your-registry.azurecr.io  # Or docker.io/username, gcr.io/project, etc.

# Clone the repository
git clone https://github.com/nathanfox/k8s-vscode-remote-debug.git
cd k8s-vscode-remote-debug/examples/fsharp-giraffe-dotnet8

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

**Debugging a Functional Handler:**

1. **Set a breakpoint** in `Program.fs` at line 23 (inside the `debugTestHandler` function)
2. **Attach debugger** by pressing F5 in VS Code
3. **Enter namespace and pod name** when prompted
4. **Port-forward the application** (in a separate terminal):
   ```bash
   ./manage.sh port-forward
   ```
5. **Trigger the endpoint:**
   ```bash
   curl http://localhost:8080/debug-test?count=3
   ```
6. **Inspect functional values:**
   - Hover over `count` to see the parsed value
   - Inspect `items` list as it's computed
   - View closure captures in lambda functions
7. **Step through pipelines:**
   - F10 (step over) for each pipeline stage
   - Watch transformations happen step-by-step
8. **Continue execution** - F5 to resume

## VS Code Extensions

**Required for debugging:**
- **Ionide-fsharp** (Ionide.ionide-fsharp) - F# language support and IntelliSense
- **C#** (ms-dotnettools.csharp) - Required for vsdbg debugger

**Recommended (helpful but not required):**
- **Kubernetes** (ms-kubernetes-tools.vscode-kubernetes-tools) - K8s cluster management

## Troubleshooting

### F# IntelliSense Not Working

Ensure Ionide-fsharp extension is installed and project dependencies are restored:
```bash
dotnet restore
```

## Example-Driven Development with AI Agents

This repository demonstrates Example-Driven Development, designed to work with AI coding assistants like [Claude Code](https://claude.ai/code).

For more on this pattern, see [Example-Driven Development Using AI Agent Claude Code](https://www.nathanfox.net/p/example-driven-development-using-ai-agent-claude-code).

**Example AI prompt:**
> "Using the k8s-vscode-remote-debug repository's F# Giraffe example, add remote debugging support to my F# web application running in Kubernetes."

The AI can generate the appropriate configuration based on the working example.

## Next Steps

For complete details including:
- Full troubleshooting guide
- Debug vs Production build configuration
- Performance considerations
- Integration patterns

See the [complete README](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/fsharp-giraffe-dotnet8/README.md) in the repository.

The repository includes examples for 8 languages/frameworks, each demonstrating the unique aspects of debugging that language in Kubernetes.
