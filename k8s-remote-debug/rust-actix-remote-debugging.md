# Debugging Rust Actix Applications in Kubernetes with Structured Tracing

Debugging Rust applications running in Kubernetes presents unique challenges. This post demonstrates debugging Rust Actix-web applications using structured logging with the `tracing` crate, a production-ready approach that works reliably with async code.

## Overview

This example from the [k8s-vscode-remote-debug](https://github.com/nathanfox/k8s-vscode-remote-debug) repository showcases:
- Building Rust applications with structured tracing
- Deploying to Kubernetes with comprehensive logging
- Using the `tracing` crate for observability
- Debugging async Rust code in production environments

**What Works:**
- ✅ Structured logging with multiple levels (trace, debug, info, warn, error)
- ✅ Function instrumentation with automatic argument logging
- ✅ Async code debugging via tracing spans
- ✅ Production-ready observability
- ✅ Low overhead when disabled
- ✅ Complex type logging with Debug trait

**Technology Stack:**
- **Language:** Rust 1.83
- **Framework:** Actix-web 4.11
- **Debugging Method:** Structured logging (`tracing` crate)
- **Runtime:** Tokio async runtime

## Why Tracing Instead of Traditional Debugging?

**TLDR**: LLDB breakpoints don't work reliably with async Rust code in Kubernetes.

Traditional debuggers like LLDB can attach to Rust processes and resolve breakpoints, but **breakpoints never trigger in async functions** running on Tokio's worker threads. This is a fundamental limitation of current debugging tools with Rust's async runtime model.

Instead, the Rust ecosystem has embraced **structured tracing** as the primary debugging and observability approach for async applications.

## How It Works

The debugging setup uses the `tracing` crate ecosystem:

1. **Tracing Subscriber:** Initialize logging backend in main()
2. **Instrumentation:** Use `#[instrument]` macro on functions
3. **Structured Logs:** Log events with structured data
4. **Spans:** Track async execution flow with tracing spans

This approach provides observability that works reliably with async code and scales to production environments.

## Key Configuration

### Application Code (main.rs)

Initialize tracing in your application:

```rust
use tracing::{info, debug, instrument};
use tracing_subscriber;

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // Initialize tracing with debug level
    tracing_subscriber::fmt()
        .with_env_filter("debug")
        .init();

    info!("Starting Rust Actix-web server on 0.0.0.0:8080");
    // ... rest of setup
}
```

### Instrumented Functions

The `#[instrument]` macro automatically logs function entry/exit with arguments:

```rust
#[instrument]
async fn debug_test(query: web::Query<DebugTestQuery>) -> Result<HttpResponse> {
    info!("Debug test called with count={}", query.count);

    let mut items = Vec::new();
    for i in 0..query.count {
        debug!("Processing item {}/{}", i + 1, query.count);
        items.push(format!("Item {}", i + 1));
        thread::sleep(StdDuration::from_millis(10));
    }

    info!("Debug test completed, returning {} items", items.len());
    Ok(HttpResponse::Ok().json(response))
}
```

See the [complete main.rs](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/rust-actix/src/main.rs) for the full application code.

### Cargo.toml Dependencies

Add tracing dependencies:

```toml
[dependencies]
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }
```

See the [complete Cargo.toml](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/rust-actix/Cargo.toml) for all dependencies.

### Dockerfile

The Dockerfile uses a multi-stage build with Rust-specific caching:

```dockerfile
# Builder stage
FROM rust:1.83 AS builder
WORKDIR /build

# Cache dependencies
COPY Cargo.toml Cargo.lock ./
RUN mkdir src && echo "fn main() {}" > src/main.rs
RUN cargo build

# Copy source and rebuild
COPY src ./src
RUN touch src/main.rs  # Force recompilation (Rust-specific cache workaround)
RUN cargo build

# Runtime stage
FROM debian:bookworm-slim
COPY --from=builder /build/target/debug/rust-actix /usr/local/bin/
EXPOSE 8080
CMD ["rust-actix"]
```

**Important:** The `touch src/main.rs` is critical - it forces Cargo to recompile and avoid stale cache artifacts. See the [complete Dockerfile](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/rust-actix/Dockerfile) for details.

## Quick Start

```bash
# Set your developer namespace and registry
export NAMESPACE=dev-yourname
export REGISTRY=your-registry.azurecr.io  # Or docker.io/username, gcr.io/project, etc.

# Clone the repository
git clone https://github.com/nathanfox/k8s-vscode-remote-debug.git
cd k8s-vscode-remote-debug/examples/rust-actix

# Build, push, and deploy
./manage.sh build
./manage.sh push
./manage.sh deploy

# Verify pod is ready
./manage.sh status
```

## Debugging Walkthrough

1. **Port-forward the application** (in a terminal):
   ```bash
   ./manage.sh port-forward
   ```

2. **Make requests to trigger tracing:**
   ```bash
   curl http://localhost:8080/health
   curl http://localhost:8080/debug-test?count=5
   curl http://localhost:8080/weatherforecast
   ```

3. **View structured logs:**
   ```bash
   ./manage.sh logs
   ```

4. **Follow logs in real-time:**
   ```bash
   ./manage.sh logs -f
   ```

### Example Output

```
2025-09-28T17:44:34.582173Z  INFO rust_actix: Starting Rust Actix-web server on 0.0.0.0:8080
2025-09-28T17:44:34.582361Z  INFO actix_server::builder: starting 1 workers
2025-09-28T17:45:12.123456Z  INFO rust_actix::debug_test: Debug test called with count=5
2025-09-28T17:45:12.123567Z DEBUG rust_actix::debug_test: Processing item 1/5
2025-09-28T17:45:12.133789Z DEBUG rust_actix::debug_test: Processing item 2/5
2025-09-28T17:45:12.143890Z DEBUG rust_actix::debug_test: Processing item 3/5
2025-09-28T17:45:12.153991Z DEBUG rust_actix::debug_test: Processing item 4/5
2025-09-28T17:45:12.164092Z DEBUG rust_actix::debug_test: Processing item 5/5
2025-09-28T17:45:12.164193Z  INFO rust_actix::debug_test: Debug test completed, returning 5 items
```

## Tracing Features

### Instrumentation Macro

The `#[instrument]` macro provides automatic function tracing:

```rust
#[instrument]
async fn my_handler(id: u32, name: String) -> Result<HttpResponse> {
    // Automatically logs function entry with: id=42, name="example"
    // Automatically logs function exit
}
```

### Structured Fields

Log structured data with key-value pairs:

```rust
info!(user_id = %user.id, action = "login", "User logged in");
debug!(count = items.len(), "Processing items");
```

### Tracing Levels

- `trace!()` - Very detailed, usually disabled
- `debug!()` - Debug information
- `info!()` - General information
- `warn!()` - Warning messages
- `error!()` - Error conditions

### Async Spans

Track async execution flow:

```rust
use tracing::instrument;

#[instrument]
async fn process_request(req: Request) -> Response {
    let span = tracing::info_span!("database_query");
    let result = async {
        // Database operations traced under this span
    }.instrument(span).await;

    result
}
```

## Benefits of Tracing Approach

**Production Ready:**
- Works reliably in production environments
- Can be enabled/disabled via environment variables
- Low overhead when disabled

**Async Compatible:**
- Designed for async Rust from the ground up
- Tracks execution across await points
- Works with Tokio, async-std, etc.

**Structured Data:**
- Log complex types with Debug trait
- Machine-parseable output
- Easy to integrate with log aggregation systems

**Zero Cost:**
- Compiled out in release builds without env filter
- No runtime overhead when disabled

## Actix-web Framework

**Actix-web** is a powerful, pragmatic Rust web framework:

- Built on Tokio async runtime
- Type-safe request routing
- Middleware support
- WebSocket support
- HTTP/2 support

Example handler:
```rust
#[get("/debug-test")]
#[instrument]
async fn debug_test(query: web::Query<DebugTestQuery>) -> Result<HttpResponse> {
    info!("Processing request with count={}", query.count);
    // Handler logic with tracing
    Ok(HttpResponse::Ok().json(response))
}
```

## Troubleshooting

### No Logs Appearing

1. **Check tracing is initialized:**
   ```rust
   tracing_subscriber::fmt()
       .with_env_filter("debug")
       .init();
   ```

2. **Verify log level:**
   - Set `RUST_LOG=debug` environment variable
   - Or use `with_env_filter("debug")` in code

3. **Check pod logs:**
   ```bash
   ./manage.sh logs --tail 100
   ```

### Binary Exits Immediately

**Problem:** Container exits with code 0 immediately after starting

**Solution:** This is usually a Cargo cache issue. The Dockerfile includes `RUN touch src/main.rs` to force recompilation. If you still see this:

```bash
# Rebuild from scratch
./manage.sh build
./manage.sh push
./manage.sh restart
```

### Missing Tracing Output

1. **Ensure functions are instrumented:**
   ```rust
   #[instrument]  // Add this
   async fn my_function() { }
   ```

2. **Check log statements exist:**
   ```rust
   info!("Important event");
   debug!("Debug details");
   ```

## Example-Driven Development with AI Agents

This repository demonstrates Example-Driven Development, designed to work with AI coding assistants like [Claude Code](https://claude.ai/code).

For more on this pattern, see [Example-Driven Development Using AI Agent Claude Code](https://www.nathanfox.net/p/example-driven-development-using-ai-agent-claude-code).

**Example AI prompt:**
> "Using the k8s-vscode-remote-debug repository's Rust Actix example, add structured tracing to my Actix-web application running in Kubernetes."

The AI can generate the appropriate tracing setup, instrumentation, and deployment configuration based on the working example.

## Next Steps

For complete details including:
- Full troubleshooting guide
- LLDB investigation details
- Advanced tracing patterns
- Production deployment considerations

See the [complete README](https://github.com/nathanfox/k8s-vscode-remote-debug/blob/develop/examples/rust-actix/README.md) in the repository.

The repository includes examples for 8 languages/frameworks, each demonstrating the unique aspects of debugging that language in Kubernetes.
