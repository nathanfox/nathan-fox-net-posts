# k8s-vscode-remote-debug: Remote Debugging Kubernetes Pods with VS Code

## Overview

[k8s-vscode-remote-debug](https://github.com/nathanfox/k8s-vscode-remote-debug) provides reference examples and working configurations for remote debugging applications running in Kubernetes pods directly from Visual Studio Code. These examples serve as practical references for developers and are designed to be used as input for AI agents like Claude Code when setting up debugging workflows.

## The Problem

Debugging applications running in Kubernetes pods has historically been challenging:
- Complex setup procedures that vary by language and framework
- Difficulty accessing debug ports and attaching debuggers
- Inconsistent workflows across different programming languages
- Time-consuming troubleshooting of distributed microservices

Developers often resort to excessive logging or struggle with complicated port-forwarding and configuration to debug issues that only manifest in Kubernetes environments.

## The Solution

k8s-vscode-remote-debug provides ready-to-use debugging configurations and examples for multiple programming languages and frameworks, enabling developers to debug Kubernetes pods as if they were running locally. The repository includes working examples for:

- **[C# .NET 8 Web API](https://www.nathanfox.net/p/remote-debugging-c-net-8-web-apis)** - Remote debugging with vsdbg using kubectl exec
- **[F# Giraffe](https://www.nathanfox.net/p/remote-debugging-f-giraffe-applications)** - Functional programming debugging with vsdbg
- **[Node.js Express](https://www.nathanfox.net/p/remote-debugging-nodejs-express-applications)** - Inspector Protocol debugging with port-forwarding
- **[Python FastAPI](https://www.nathanfox.net/p/remote-debugging-python-fastapi-applications)** - Async debugging with debugpy
- **[Go Gin](https://www.nathanfox.net/p/remote-debugging-go-gin-applications)** - Delve debugger with goroutine inspection
- **[Java Spring Boot](https://www.nathanfox.net/p/remote-debugging-java-spring-boot)** - JDWP debugging for enterprise Java
- **[Rust Actix](https://www.nathanfox.net/p/debugging-rust-actix-applications)** - Structured tracing for async Rust
- **[Elixir Phoenix](https://www.nathanfox.net/p/remote-debugging-elixir-phoenix-applications)** - Remote - Kubernetes extension for in-pod debugging

Each blog post provides detailed setup instructions, troubleshooting guides, and language-specific debugging techniques.

Key features:
- **Per-developer namespace isolation**: Each developer works in their own namespace
- **Unified management scripts**: Consistent commands across all language examples
- **Full debugging capabilities**: Set breakpoints, inspect variables, step through code
- **Language-specific configurations**: Optimized setups for each supported language

## Use Cases

### Microservice Troubleshooting
Debug a specific microservice in your Kubernetes cluster while it interacts with other services in real-time.

### Environment-Specific Issues
Investigate bugs that only occur in Kubernetes environments with specific configurations, resource constraints, or service interactions.

### Active Development in Kubernetes
Develop directly in Kubernetes environments to ensure your code works correctly with real service dependencies, configurations, and resource constraints from the start. Debug and iterate on your code while it runs in the actual deployment environment.

### AI-Assisted Setup for Your Own Projects
Use this repository as a reference source for AI agents like Claude Code to configure remote debugging in your own projects. The examples provide complete, working patterns that AI agents can adapt to your specific codebase and requirements.

**Example prompts for AI agents:**
- "Using k8s-vscode-remote-debug as a reference, set up remote debugging for my FastAPI application"
- "Configure my Node.js app for Kubernetes debugging following the patterns in k8s-vscode-remote-debug"
- "Add VS Code remote debugging to my Go service similar to the go-gin example in k8s-vscode-remote-debug"

The AI agent will:
1. Analyze the relevant example from this repository
2. Adapt the Dockerfile to include debugging tools for your application
3. Generate appropriate Kubernetes manifests with debug ports
4. Create VS Code launch configurations
5. Provide a customized `manage.sh` script for your workflow

### Development Workflow
Combine with tools like [nginx-dev-gateway](https://github.com/nathanfox/nginx-dev-gateway) to create a complete development environment where you can route traffic to debug-enabled pods and step through code execution.

## Getting Started

The repository provides complete working examples for each supported language. For setup instructions, configuration details, and language-specific debugging guides, see the [k8s-vscode-remote-debug repository](https://github.com/nathanfox/k8s-vscode-remote-debug).

Each example includes:
- Dockerfile with debugging tools
- Kubernetes manifests
- VS Code launch configurations
- Step-by-step setup instructions

## Architecture

The debugging setup typically involves:
1. Building a debug-enabled container image with debugging tools
2. Deploying the application to a Kubernetes pod
3. Forwarding the debug port to your local machine
4. Configuring VS Code to attach to the remote debugger

The repository's `manage.sh` script automates much of this process, providing commands to build, deploy, and connect to debug-enabled pods with minimal manual configuration.

## Example Workflow

Here's a typical workflow using the `manage.sh` script:

```bash
# Set your developer namespace and registry
export NAMESPACE=dev-yourname
export REGISTRY=your-registry.azurecr.io

# Navigate to an example (e.g., Python FastAPI)
cd examples/python-fastapi

# Build and push the debug-enabled image
./manage.sh build
./manage.sh push

# Deploy to your Kubernetes namespace
./manage.sh deploy

# Start port forwarding for debugging
./manage.sh debug

# Now open VS Code and press F5 to attach debugger
# Set breakpoints and debug as if running locally

# Clean up when done
./manage.sh delete
```

The unified `manage.sh` script works consistently across all language examples, simplifying the debugging workflow regardless of the technology stack.