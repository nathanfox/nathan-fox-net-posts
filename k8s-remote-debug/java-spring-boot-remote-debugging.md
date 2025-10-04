# Remote Debugging Java Spring Boot Applications in Kubernetes with VS Code

Debugging Java applications running in Kubernetes is straightforward using JDWP (Java Debug Wire Protocol). This post demonstrates remote debugging of Java Spring Boot applications using JDWP and VS Code, allowing you to debug enterprise Java applications running in Kubernetes.

## Overview

This example from the [k8s-vscode-remote-debug](https://github.com/nathanfox/k8s-vscode-remote-debug) repository showcases:
- Building Docker images with JDWP enabled
- Deploying Spring Boot to Kubernetes with debug configuration
- Port-forwarding debug port (5005) from local machine to remote pod
- Attaching VS Code debugger using JDWP
- Debugging Spring Boot applications with full breakpoint support

**What Works:**
- ✅ Breakpoints in Java files
- ✅ Variable inspection (primitives, objects, collections)
- ✅ Call stack navigation
- ✅ Conditional breakpoints
- ✅ Expression evaluation in Debug Console
- ✅ Watch expressions
- ✅ Exception breakpoints
- ✅ Hot code replace (limited)
- ✅ Thread inspection

**Technology Stack:**
- **Language:** Java 21 LTS
- **Framework:** Spring Boot 3.2.0
- **Debugger:** JDWP (Java Debug Wire Protocol)
- **Debug Method:** Port-forward to debug port 5005

## How It Works

The debugging setup uses JDWP, Java's standard debugging protocol:

1. **JVM Configuration:** Enable JDWP via `-agentlib:jdwp` JVM argument
2. **Debug Server:** JVM listens on port 5005 for debugger connections
3. **Port-Forwarding:** VS Code connects via localhost:5005
4. **JDWP Protocol:** Standard Java debugging over TCP

JDWP is enabled through the `JAVA_OPTS` environment variable, making it easy to toggle debugging on/off without rebuilding the image.

## Key Configuration

### Dockerfile (Debug Mode)

The Dockerfile uses a multi-stage Maven build:

```dockerfile
# Build stage - Maven compilation
FROM maven:3.9-eclipse-temurin-21 AS builder
WORKDIR /build
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage - Eclipse Temurin JRE
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /build/target/*.jar app.jar
EXPOSE 8080 5005
ENTRYPOINT exec java $JAVA_OPTS -jar app.jar
```

The `JAVA_OPTS` environment variable allows runtime configuration of JDWP. See the [complete Dockerfile](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/java-spring-boot/Dockerfile) for the full build.

### Kubernetes Deployment

JDWP is enabled via environment variable in the deployment:

```yaml
env:
- name: JAVA_OPTS
  value: "-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005 -Xmx384m"
```

**JDWP Parameters:**
- `transport=dt_socket` - Use TCP socket transport
- `server=y` - JVM acts as debug server
- `suspend=n` - Don't wait for debugger on startup
- `address=*:5005` - Listen on all interfaces, port 5005
- `-Xmx384m` - Limit heap to 384MB (75% of 512Mi container limit)

See the [complete deployment.yaml](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/java-spring-boot/k8s/deployment.yaml) for full configuration including resource limits and health checks.

### VS Code launch.json

The debugger configuration attaches to JDWP:

```json
{
  "name": "Attach to Remote Pod",
  "type": "java",
  "request": "attach",
  "hostName": "localhost",
  "port": 5005,
  "projectName": "demo"
}
```

Key settings:
- `type: "java"` - Java debugging
- `port: 5005` - JDWP default debug port
- `projectName: "demo"` - Spring Boot project name

Unlike Go or Node.js, Java debugging does **not require path mapping** - the Java extension automatically handles source file resolution.

See the [complete launch.json](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/java-spring-boot/.vscode/launch.json) for all options.

## Quick Start

```bash
# Set your developer namespace and registry
export NAMESPACE=dev-yourname
export REGISTRY=your-registry.azurecr.io  # Or docker.io/username, gcr.io/project, etc.

# Clone the repository
git clone https://github.com/nathanfox/k8s-vscode-remote-debug.git
cd k8s-vscode-remote-debug/examples/java-spring-boot

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

1. **Set a breakpoint** in `DebugTestController.java` at line 28 (inside the loop)
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
   - Check Variables panel to see `items`, `i`, request parameters
   - Inspect Spring Boot context and beans
8. **Step through code:**
   - F10 (step over) to execute current line
   - F11 (step into) to step into methods
   - Watch the `items` list grow as you step through the loop
9. **Continue execution** - F5 to resume

## Debugging Spring Boot Features

### Spring Boot Controllers

Example controller debugging:

```java
@RestController
public class DebugTestController {

    @GetMapping("/debug-test")
    public DebugTestResponse debugTest(
            @RequestParam(defaultValue = "3") int count) {

        // Breakpoint here - inspect request parameters
        if (count < 1 || count > 20) {
            count = 3;
        }

        List<String> items = new ArrayList<>();
        for (int i = 0; i < count; i++) {  // Breakpoint in loop
            items.add("Item " + (i + 1));
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }

        return new DebugTestResponse(
            count,
            items,
            Instant.now().toString()
        );
    }
}
```

Set breakpoints to inspect:
- Spring MVC request parameters
- Response object construction
- Exception handling paths

### JVM Memory Management

The deployment includes JVM memory configuration:

```yaml
- name: JAVA_OPTS
  value: "-agentlib:jdwp=... -Xmx384m"
```

**Important:** Always set `-Xmx` to ~75% of container memory limit to account for non-heap memory (stack, metaspace, code cache).

## VS Code Extensions

**Required for debugging:**
- **Extension Pack for Java** (vscjava.vscode-java-pack) - Java language support
- **Debugger for Java** (vscjava.vscode-java-debug) - Java debugging support
- **Maven for Java** (vscjava.vscode-maven) - Maven project support

**Recommended (helpful but not required):**
- **Kubernetes** (ms-kubernetes-tools.vscode-kubernetes-tools) - K8s cluster management

The Extension Pack for Java includes the debugger and Maven extensions, so installing it covers all requirements.

## Troubleshooting

### Debugger Won't Attach

If VS Code shows connection errors:

1. **Verify port-forward is running:**
   ```bash
   lsof -i :5005
   ```

2. **Check pod is running:**
   ```bash
   ./manage.sh status
   ```

3. **Verify JDWP is listening:**
   ```bash
   ./manage.sh logs | grep -i "Listening for transport"
   ```

4. **Check JAVA_OPTS environment variable:**
   ```bash
   kubectl get deployment java-spring-boot -n $NAMESPACE -o yaml | grep JAVA_OPTS
   ```

### Breakpoints Not Hitting

1. **Verify source code matches deployed version:**
   - Ensure you've rebuilt and redeployed after code changes
   - Check `./manage.sh status` shows the correct image

2. **Check breakpoint is on executable line:**
   - VS Code shows red circle for active breakpoints
   - Gray circle means breakpoint couldn't bind

3. **Verify request reaches the endpoint:**
   ```bash
   ./manage.sh logs --follow
   # Make request in another terminal
   curl http://localhost:8080/debug-test
   ```

### Pod OOMKilled (Out Of Memory)

If the pod restarts with exit code 137:

1. **Increase container memory limit:**
   ```yaml
   resources:
     limits:
       memory: "768Mi"  # Increased from 512Mi
   ```

2. **Adjust JVM heap proportionally:**
   ```yaml
   - name: JAVA_OPTS
     value: "-agentlib:jdwp=... -Xmx576m"  # 75% of 768Mi
   ```

## Enabling/Disabling Debug Mode

To toggle debugging without rebuilding:

**Enable debugging:**
```bash
kubectl set env deployment/java-spring-boot \
  JAVA_OPTS="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005 -Xmx384m" \
  -n $NAMESPACE
```

**Disable debugging:**
```bash
kubectl set env deployment/java-spring-boot \
  JAVA_OPTS="-Xmx384m" \
  -n $NAMESPACE
```

## Example-Driven Development with AI Agents

This repository demonstrates Example-Driven Development, designed to work with AI coding assistants like [Claude Code](https://claude.ai/code).

For more on this pattern, see [Example-Driven Development Using AI Agent Claude Code](https://www.nathanfox.net/p/example-driven-development-using-ai-agent-claude-code).

**Example AI prompt:**
> "Using the k8s-vscode-remote-debug repository's Java Spring Boot example, add remote debugging support to my Spring Boot application running in Kubernetes."

The AI can generate the appropriate JDWP configuration, Dockerfile with multi-stage Maven build, and deployment manifests based on the working example.

## Next Steps

For complete details including:
- Full troubleshooting guide
- Spring Boot Actuator endpoints
- Hot code replace limitations
- Production debugging considerations

See the [complete README](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/java-spring-boot/README.md) in the repository.

The repository includes examples for 8 languages/frameworks, each demonstrating the unique aspects of debugging that language in Kubernetes.
