# Remote Debugging Go Gin Applications in Kubernetes with VS Code

Debugging Go applications running in Kubernetes is powerful with Delve, the Go debugger. This post demonstrates remote debugging of Go Gin applications using Delve and VS Code, allowing you to debug goroutines, inspect interfaces, and step through your application running in Kubernetes.

## Overview

This example from the [k8s-vscode-remote-debug](https://github.com/nathanfox/k8s-vscode-remote-debug) repository showcases:
- Building Docker images with Delve debugger enabled
- Deploying to Kubernetes with debug port exposed
- Port-forwarding debug port (2345) from local machine to remote pod
- Attaching VS Code debugger using Delve
- Debugging goroutines and concurrent code

**What Works:**
- ✅ Breakpoints in Go files
- ✅ Variable inspection (primitives, structs, slices, maps)
- ✅ Call stack navigation
- ✅ Conditional breakpoints
- ✅ Expression evaluation in Debug Console
- ✅ Watch expressions
- ✅ Goroutine inspection and switching
- ✅ Panic stack traces
- ✅ Interface value inspection

**Technology Stack:**
- **Language:** Go (v1.23)
- **Framework:** Gin
- **Debugger:** Delve (dlv)
- **Debug Method:** Port-forward to debug port 2345

## How It Works

The debugging setup uses Delve, the standard Go debugger:

1. **Docker Build:** Compile Go with debug symbols (`-gcflags="all=-N -l"`)
2. **Delve Wrapper:** Run application under Delve in headless mode
3. **Debug Adapter Protocol:** Delve exposes DAP on port 2345
4. **Port-Forwarding:** VS Code connects via localhost:2345

This approach wraps the Go application with Delve, which manages the debug session and communicates with VS Code.

## Key Configuration

### Dockerfile (Debug Mode)

The Dockerfile compiles Go with debug symbols and runs under Delve:

```dockerfile
# Build stage - compile with debug symbols
RUN go build -gcflags="all=-N -l" -o app main.go
RUN go install github.com/go-delve/delve/cmd/dlv@latest

# Runtime stage - run under Delve
CMD ["/usr/local/bin/dlv", "--listen=:2345", "--headless=true", \
     "--api-version=2", "--accept-multiclient", "exec", "--continue", "/app/app"]
```

Key Delve flags:
- `--listen=:2345` - Listen on port 2345
- `--headless=true` - Run without interactive terminal
- `--api-version=2` - Use DAP for VS Code compatibility
- `--accept-multiclient` - Allow multiple debugger connections
- `--continue` - Start the app immediately

See the [complete Dockerfile](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/go-gin/Dockerfile) for the full multi-stage build.

### VS Code launch.json

The debugger configuration attaches to Delve's DAP server:

```json
{
  "name": "Attach to Remote Pod",
  "type": "go",
  "request": "attach",
  "mode": "remote",
  "port": 2345,
  "host": "localhost",
  "substitutePath": [
    {
      "from": "${workspaceFolder}",
      "to": "/build"
    }
  ]
}
```

Key settings:
- `mode: "remote"` - Remote debugging mode
- `port: 2345` - Delve's default debug port
- `substitutePath` - Map local source to container build path

See the [complete launch.json](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/go-gin/.vscode/launch.json) for all options.

## Quick Start

```bash
# Set your developer namespace and registry
export NAMESPACE=dev-yourname
export REGISTRY=your-registry.azurecr.io  # Or docker.io/username, gcr.io/project, etc.

# Clone the repository
git clone https://github.com/nathanfox/k8s-vscode-remote-debug.git
cd k8s-vscode-remote-debug/examples/go-gin

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

1. **Set a breakpoint** in `main.go` at line 85 (inside the `/debug-test` handler)
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
6. **Breakpoint hits** - execution pauses at line 85
7. **Inspect variables:**
   - Hover over `count` to see the value (3)
   - Check Variables panel to see `items`, `i`
   - Inspect `c` (Gin context) - view request data
8. **Step through code:**
   - F10 (step over) to execute current line
   - F11 (step into) to step into functions
   - Watch the `items` slice grow as you step through the loop
9. **Continue execution** - F5 to resume

## Debugging Go-Specific Features

### Goroutine Debugging

View and debug concurrent goroutines:

- **Goroutine Panel:** See all running goroutines in VS Code
- **Switch Goroutines:** Click to switch between goroutines
- **Goroutine States:** See which are running, blocked, or waiting
- **Goroutine-Local Variables:** Inspect variables in each goroutine's scope

### Interface Debugging

Go interfaces are inspectable:
- See the concrete type behind an interface
- Use `reflect.TypeOf()` in Debug Console to check types
- Inspect interface method sets

### Build Flags for Debugging

The `-gcflags="all=-N -l"` flags are critical:
- `-N` - Disables optimizations (variables aren't optimized away)
- `-l` - Disables function inlining (breakpoints work reliably)

Without these flags, variables may show as `<optimized out>` and breakpoints may not hit.

## Gin Framework Debugging

**Gin** is a high-performance HTTP framework for Go:

Example handler debugging:
```go
r.GET("/debug-test", func(c *gin.Context) {
    countStr := c.DefaultQuery("count", "3")  // Breakpoint here
    count, err := strconv.Atoi(countStr)
    if err != nil || count < 1 || count > 20 {
        count = 3
    }

    items := make([]string, 0, count)
    for i := 0; i < count; i++ {  // Breakpoint in loop
        items.append(items, "Item "+strconv.Itoa(i+1))
        time.Sleep(10 * time.Millisecond)
    }

    c.JSON(http.StatusOK, DebugTestResponse{
        Count:     count,
        Items:     items,
        Timestamp: time.Now().UTC().Format(time.RFC3339),
    })
})
```

Set breakpoints to inspect Gin context, query parameters, and response building.

## VS Code Extensions

**Required for debugging:**
- **Go** (golang.go) - Go language support and debugging

**Recommended (helpful but not required):**
- **Kubernetes** (ms-kubernetes-tools.vscode-kubernetes-tools) - K8s cluster management

The Go extension includes Delve integration, so no additional debugger extensions are needed.

## Troubleshooting

### Debugger Won't Connect

If VS Code shows "Cannot connect to runtime process":

1. **Verify port-forward is running:**
   ```bash
   ps aux | grep "port-forward.*2345"
   ```

2. **Check pod is running:**
   ```bash
   ./manage.sh status
   ```

3. **Verify Delve is listening:**
   ```bash
   ./manage.sh logs | grep "API server listening"
   ```

4. **Restart port-forward:**
   ```bash
   pkill -f "port-forward.*2345"
   ./manage.sh debug
   ```

### Breakpoints Not Hitting

1. **Ensure binary compiled with debug symbols:**
   - Check Dockerfile uses `-gcflags="all=-N -l"`
   - Rebuild: `./manage.sh build && ./manage.sh push && ./manage.sh restart`

2. **Verify source path mapping:**
   - `substitutePath` must map `${workspaceFolder}` to `/build`
   - This is where source files were during compilation

3. **Check Delve is attached:**
   - Debug Console should show connection message
   - VS Code status bar shows "Go: Delve [running]"

### Variables Show `<optimized out>`

**Problem:** Variables appear as `<optimized out>` in the debugger

**Solution:** Ensure you're using `-gcflags="all=-N -l"` in the build. These flags disable optimizations that remove debug information.

## Example-Driven Development with AI Agents

This repository demonstrates Example-Driven Development, designed to work with AI coding assistants like [Claude Code](https://claude.ai/code).

For more on this pattern, see [Example-Driven Development Using AI Agent Claude Code](https://www.nathanfox.net/p/example-driven-development-using-ai-agent-claude-code).

**Example AI prompt:**
> "Using the k8s-vscode-remote-debug repository's Go Gin example, add remote debugging support to my Gin application running in Kubernetes."

The AI can generate the appropriate Dockerfile with Delve setup, launch.json configuration, and deployment manifests based on the working example.

## Next Steps

For complete details including:
- Full troubleshooting guide
- Memory profiling with pprof
- Performance debugging tips
- Delve CLI commands reference

See the [complete README](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/go-gin/README.md) in the repository.

The repository includes examples for 8 languages/frameworks, each demonstrating the unique aspects of debugging that language in Kubernetes.
