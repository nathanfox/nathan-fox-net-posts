# Remote Debugging Elixir Phoenix Applications in Kubernetes with VS Code

Debugging Elixir applications in Kubernetes is challenging. After extensive experimentation with various approaches, this post demonstrates the solution that actually works: using VS Code's Remote - Kubernetes extension to run ElixirLS inside the pod.

## Overview

This example from the [k8s-vscode-remote-debug](https://github.com/nathanfox/k8s-vscode-remote-debug) repository showcases:
- Building Elixir Phoenix applications with debug support
- Using Remote - Kubernetes extension for in-pod debugging
- Running ElixirLS inside the Kubernetes pod
- Full breakpoint debugging with proper module interpretation

**What Works:**
- ✅ Breakpoints in Elixir files
- ✅ Variable inspection (atoms, lists, maps, structs)
- ✅ Call stack navigation
- ✅ Step debugging (step over, step into, continue)
- ✅ Watch expressions
- ✅ Module interpretation with memory-optimized configuration
- ✅ Phoenix framework debugging

**Technology Stack:**
- **Language:** Elixir 1.18
- **Framework:** Phoenix
- **Debugger:** ElixirLS (running inside pod)
- **Debug Method:** Remote - Kubernetes extension (SSH to pod)

## The Challenge: Why This Was Difficult

**This example took significant effort to get working.** Several approaches were attempted:

### ❌ Approaches That Didn't Work

1. **ElixirLS Remote Attach** - The Erlang `:int` module (used for debugging) can't interpret modules that are already loaded, making traditional remote attach impossible.

2. **IEx.pry** - Requires an interactive TTY, which is incompatible with containerized environments.

3. **Port-forwarding to debugpy/DAP** - Erlang's distributed nature and module loading order prevented reliable debugging.

### ✅ The Solution: Remote - Kubernetes Extension

After extensive testing, the working solution runs ElixirLS **inside the pod** using VS Code's Remote - Kubernetes extension. This:
- Avoids distributed Erlang complexity
- Runs the debugger in the same environment as the app
- Provides full VS Code integration
- Works reliably with proper module interpretation

## How It Works

The debugging setup uses VS Code's remote development capabilities:

1. **SSH Connection:** VS Code connects to the pod via SSH
2. **In-Pod ElixirLS:** ElixirLS runs inside the pod's environment
3. **Module Interpretation:** Only specified modules are interpreted for debugging
4. **Memory Optimization:** Framework modules are excluded to reduce memory usage

This approach sidesteps the limitations of remote attach by running everything in the same Erlang VM.

## Key Configuration

### Dockerfile

The Dockerfile includes SSH server and development tools:

```dockerfile
# SSH for Remote - Kubernetes
RUN apk add --no-cache openssh bash curl git

# Configure SSH
RUN mkdir -p /run/sshd && \
    ssh-keygen -A

# VS Code config embedded in image
COPY .vscode-remote /app/.vscode-remote

# Start IEx
CMD iex --name ${RELEASE_NODE} --cookie ${RELEASE_COOKIE} -S mix
```

See the [complete Dockerfile](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/elixir-phoenix/Dockerfile) for full configuration.

### Embedded VS Code Configuration

The `.vscode-remote/launch.json` is embedded in the Docker image:

```json
{
  "type": "mix_task",
  "name": "mix phx.server",
  "request": "launch",
  "task": "phx.server",
  "projectDir": "${workspaceRoot}",
  "debugAutoInterpretAllModules": false,
  "debugInterpretModulesPatterns": [
    "^Elixir\\.ElixirPhoenixWeb\\.ApiController$"
  ],
  "excludeModules": [
    "^Elixir\\.Phoenix\\..*",
    "^Elixir\\.Plug\\..*",
    "^Elixir\\.Bandit\\..*"
  ],
  "requireFiles": [
    "lib/elixir_phoenix_web/controllers/api_controller.ex"
  ]
}
```

**Key settings:**
- `debugAutoInterpretAllModules: false` - Memory optimization
- `debugInterpretModulesPatterns` - Only interpret modules you want to debug
- `excludeModules` - Exclude Phoenix framework modules

See the [complete launch.json](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/elixir-phoenix/.vscode-remote/launch.json) for all patterns.

### Kubernetes Deployment

The deployment includes necessary ports and secrets:

```yaml
env:
- name: RELEASE_NODE
  value: "elixir_phoenix@127.0.0.1"
- name: RELEASE_COOKIE
  valueFrom:
    secretKeyRef:
      name: elixir-erlang-cookie
      key: cookie

ports:
- containerPort: 4000
  name: http
- containerPort: 22
  name: ssh
```

See the [complete deployment.yaml](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/elixir-phoenix/k8s/deployment.yaml) for full configuration.

## Quick Start

```bash
# Set your developer namespace and registry
export NAMESPACE=dev-yourname
export REGISTRY=your-registry.azurecr.io  # Or docker.io/username, gcr.io/project, etc.

# Clone the repository
git clone https://github.com/nathanfox/k8s-vscode-remote-debug.git
cd k8s-vscode-remote-debug/examples/elixir-phoenix

# Build, push, and deploy
./manage.sh build
./manage.sh push
./manage.sh deploy

# Verify pod is ready
./manage.sh status
```

## Debugging Walkthrough

### Step 1: Install Remote - Kubernetes Extension

In VS Code:
- Open Extensions (Cmd+Shift+X / Ctrl+Shift+X)
- Search for "Remote - Kubernetes"
- Install **Remote - Kubernetes** by Okteto

### Step 2: Connect to the Pod

**Via Kubernetes Sidebar (easiest):**
1. Open Kubernetes view in VS Code sidebar
2. Expand your cluster → Workloads → Pods
3. Right-click on `elixir-phoenix-*` pod
4. Select **"Attach Visual Studio Code"**
5. VS Code will reload and connect to the pod via SSH

**Via Command Palette:**
1. Open Command Palette (Cmd+Shift+P / Ctrl+Shift+P)
2. Run: `Dev Containers: Attach to Running Kubernetes Container...`
3. Select your namespace and the `elixir-phoenix-*` pod
4. Select container: `elixir-phoenix`

### Step 3: Install ElixirLS in Remote Session

- VS Code will prompt to install recommended extensions
- Click "Install" for ElixirLS extension
- Wait for ElixirLS to compile and start (this may take a few minutes)

### Step 4: Open App Directory

- In the remote VS Code window, open folder: `/app`
- ElixirLS will automatically load the embedded launch.json

### Step 5: Set Breakpoints

- Open `lib/elixir_phoenix_web/controllers/api_controller.ex`
- Click in the gutter to set a breakpoint (e.g., line 17 in `debug_test/2`)

### Step 6: Start Debugging

1. Open Run and Debug panel (Cmd+Shift+D / Ctrl+Shift+D)
2. Select "mix phx.server" configuration
3. Press F5 to start debugging
4. Phoenix will compile and start with debugging enabled

### Step 7: Trigger the Endpoint

In your **local terminal** (not the remote session):

```bash
# Port-forward the application
./manage.sh port-forward

# Make a request in another terminal
curl http://localhost:4000/debug-test?count=3
```

### Step 8: Debug

- Execution pauses at your breakpoint
- Inspect variables in Variables panel
- Use step controls:
  - F10 - Step over
  - F11 - Step into
  - F5 - Continue

## Phoenix Framework Debugging

**Phoenix** is Elixir's most popular web framework:

Example controller debugging:

```elixir
defmodule ElixirPhoenixWeb.ApiController do
  use ElixirPhoenixWeb, :controller

  def debug_test(conn, %{"count" => count_str}) do
    count = String.to_integer(count_str)  # Breakpoint here

    items = Enum.map(1..count, fn i ->
      "Item #{i}"  # Watch items build
    end)

    json(conn, %{
      count: count,
      items: items,
      timestamp: DateTime.utc_now()
    })
  end
end
```

Set breakpoints to inspect:
- Phoenix connection (`conn`) struct
- Request parameters
- Response data

## Memory Optimization

The launch.json includes critical memory optimizations:

```json
"debugAutoInterpretAllModules": false,
"debugInterpretModulesPatterns": [
  "^Elixir\\.ElixirPhoenixWeb\\.ApiController$"
],
"excludeModules": [
  "^Elixir\\.Phoenix\\..*",
  "^Elixir\\.Plug\\..*"
]
```

**Why this matters:**
- Interpreting all modules causes excessive memory usage
- Phoenix framework has many modules
- Only interpret the specific modules you're debugging

## VS Code Extensions

**Required for debugging:**
- **Remote - Kubernetes** (okteto.remote-kubernetes) - Connects VS Code to pod
- **ElixirLS** (jakebecker.elixir-ls) - Elixir language support and debugger
- **Kubernetes** (ms-kubernetes-tools.vscode-kubernetes-tools) - Cluster navigation

The Kubernetes extension helps you find and attach to pods, while Remote - Kubernetes handles the SSH connection.

## Troubleshooting

### Can't Connect to Pod

1. **Verify pod is running:**
   ```bash
   ./manage.sh status
   ```

2. **Check SSH is accessible:**
   ```bash
   kubectl exec -n $NAMESPACE -l app=elixir-phoenix -- ssh -V
   ```

3. **Verify Remote - Kubernetes extension is installed**

### ElixirLS Won't Start

1. **Check Elixir/Erlang versions in pod:**
   ```bash
   kubectl exec -n $NAMESPACE -l app=elixir-phoenix -- elixir --version
   ```

2. **Rebuild if needed:**
   ```bash
   ./manage.sh build
   ./manage.sh push
   ./manage.sh restart
   ```

### Breakpoints Not Hitting

1. **Verify module is in debugInterpretModulesPatterns:**
   - Check `.vscode-remote/launch.json` in the pod
   - Add your module pattern if missing

2. **Check module is compiled:**
   - In the debug terminal, verify Phoenix compiled successfully

3. **Restart debug session:**
   - Stop debugger (Shift+F5)
   - Start again (F5)

### High Memory Usage

If the pod runs out of memory:

1. **Reduce interpreted modules:**
   - Be more specific in `debugInterpretModulesPatterns`
   - Add more patterns to `excludeModules`

2. **Increase pod memory limit** in deployment.yaml

## Example-Driven Development with AI Agents

This repository demonstrates Example-Driven Development, designed to work with AI coding assistants like [Claude Code](https://claude.ai/code).

For more on this pattern, see [Example-Driven Development Using AI Agent Claude Code](https://www.nathanfox.net/p/example-driven-development-using-ai-agent-claude-code).

**Example AI prompt:**
> "Using the k8s-vscode-remote-debug repository's Elixir Phoenix example, add remote debugging support to my Phoenix application running in Kubernetes using the Remote - Kubernetes extension."

The AI can generate the appropriate Dockerfile with SSH setup, embedded VS Code configuration, and deployment manifests based on the working example.

## Next Steps

For complete details including:
- Full troubleshooting guide
- Alternative debugging approaches
- Manual in-pod debugging method
- Implementation planning notes

See the [complete README](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/elixir-phoenix/README.md) in the repository.

The repository includes examples for 8 languages/frameworks, each demonstrating the unique aspects of debugging that language in Kubernetes.
