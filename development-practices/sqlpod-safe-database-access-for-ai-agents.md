# sqlpod: Safe Database Access for AI Agents with Nothing but kubectl exec

**Source:** [GitHub (MIT)](https://github.com/nathanfox/sqlpod) | **Image:** `ghcr.io/nathanfox/sqlpod` (multi-arch, amd64/arm64)

---

## The Problem: Agents Need the Database, Not the Credentials

AI coding agents are genuinely good at SQL. Hand one a schema question — "how many orders are stuck in this state?", "does this column actually contain what the code assumes?" — and it will write the query faster than you can.

The hard part isn't the SQL. It's the access. The interesting databases live inside a Kubernetes cluster, reachable from the pods but not from your laptop. Every conventional way to bridge that gap makes me uncomfortable in the same way: a port-forward, a bastion pod with `psql` installed, an MCP database server, an internal HTTP endpoint — each one either stands up a new network endpoint that now needs its own authentication, or puts a connection string somewhere the agent can see it. And anything an agent sees ends up in a transcript.

So I built [sqlpod](https://github.com/nathanfox/sqlpod): a tiny Go binary that idles in a pod in your developer namespace, runs ad-hoc SQL against databases only the cluster can reach, and hands back one JSON document per query. No server, no port, no token. The transport is `kubectl exec`, which means access control is exactly your kubeconfig's RBAC to the namespace — nothing new to secure.

## What It Does

The flow is short. You put a connection string in a Kubernetes secret, deploy the pod once, and then query it all day:

```bash
./manage.sh set-conn "postgres://reader:pass@pg:5432/mydb?sslmode=require"
./manage.sh deploy

./query.sh query "SELECT id, name FROM customers LIMIT 5"
```

Every query returns a single JSON document built for programmatic consumption:

```json
{"columns":["id","name"],"durationMs":12,"maxRows":1000,"mode":"read",
 "rowCount":2,"rows":[[1,"a"],[2,"b"]],"truncated":false}
```

It supports SQL Server, PostgreSQL, and MySQL, and the engine is inferred from the connection string's scheme — no configuration beyond the DSN itself. A single pod can also serve several databases via named connections (`--conn orders`, `--conn warehouse`), each with its own read and optional write credentials; `deploy` discovers them from the secret's key names, so the list never has to be maintained anywhere else.

Read-only is the default. Writes require an explicit `--write` flag *and* a separate write connection string in the secret — if that key isn't set, `--write` fails cleanly rather than silently writing through the read connection.

## The Trick: kubectl exec Instead of a Server

This is the part of the design I find most satisfying. The pod's entrypoint is `/sqlpod idle`, and the entire "server" is this:

```go
if len(os.Args) > 1 && os.Args[1] == "idle" {
    // Container entrypoint: stay alive so the pod is warm for `kubectl exec`.
    // The distroless base image has no `sleep`, so the binary idles itself.
    // Block on a signal rather than `select {}` — an empty select trips Go's
    // deadlock detector and aborts. This also gives a clean SIGTERM shutdown
    // when the pod is deleted.
    sig := make(chan os.Signal, 1)
    signal.Notify(sig, syscall.SIGINT, syscall.SIGTERM)
    <-sig
    return
}
```

The idle process never runs a query. Its only job is to keep the pod — and therefore the secret-injected environment and the network position inside the cluster — alive and warm.

Each query is a *fresh* `kubectl exec` of the same binary:

```bash
kubectl exec -n "$NAMESPACE" "$pod" -- /sqlpod "SELECT ..."
```

The exec'd process inherits the container's environment, so it finds the connection string in `SQLPOD_CONN` (injected from the secret via `secretKeyRef`) without the idle process being involved at all. It connects, runs the query inside a transaction, prints JSON to stdout, and exits. `query.sh` never allocates a TTY, so the JSON comes back clean.

What falls out of this shape for free is the interesting part:

- **No listener, no attack surface.** There is no port to scan, no endpoint to authenticate, no token to rotate. If you can't `kubectl exec` into the namespace, you can't query.
- **No startup latency.** The pod is already warm; a query costs one process spawn, not a container start.
- **Auditing already exists.** Kubernetes audit logging covers `exec` calls — query access is visible in a system you already run.

The binary itself has zero Kubernetes dependency — it reads env vars and speaks argv/stdin/stdout. `kubectl exec` just happens to be a very good transport for that contract.

## Security Posture

### Credentials never touch the client

The connection string lives in the secret and reaches the binary as an environment variable — it is never a command-line argument, so it can't leak into process listings or shell history, and the machine running the agent never holds it at all. Error messages scrub both the original DSN and the driver-normalized form, and a test named `TestErrorsNeverContainPassword` keeps it that way. An agent's transcript can't capture what the agent never sees.

### Read-only, enforced twice

Layer one is a genuinely read-only login (`db_datareader`, `pg_read_all_data`, or `SELECT`-only) in the secret's `conn-string` key. Layer two is in the engine: read-mode statements run inside a transaction that is always rolled back, and on PostgreSQL and MySQL the transaction is additionally opened `READ ONLY` so the server itself rejects writes. One honest caveat the README spells out: MySQL DDL auto-commits and escapes the transaction, so on MySQL the read-only login is the only thing standing between read mode and a `DROP TABLE`. Don't put a DDL-capable login in the read key.

### A hardened pod

The image is `gcr.io/distroless/static:nonroot` — no shell, no package manager, a single static binary. The deployment sets `runAsNonRoot`, `readOnlyRootFilesystem`, `allowPrivilegeEscalation: false`, drops all capabilities, and — since the binary never talks to the Kubernetes API — doesn't mount a service-account token.

### Least privilege by file boundary

Queries go through `query.sh`; lifecycle and credentials go through `manage.sh`. `query.sh` has no deploy, delete, or secret commands to reach, and it whitelists the flags it forwards — the in-pod binary accepts `--file` for reading SQL from a path, but `query.sh` deliberately doesn't pass it through.

## Using It from an AI Agent

The two-script split exists precisely so an agent's standing permissions can be scoped tightly. For unattended use — background agents, CI, auto-approved allowlists — a Claude Code allowlist needs exactly one entry:

```
Bash(./query.sh *)
```

Its worst case is running SQL, which is already governed by the read-only connection and your namespace RBAC. In interactive sessions where you approve each command, there's no harm letting the agent drive `manage.sh` too — deploys and secret changes are visible and gated per call.

From the agent's side, usage is one shell call in, one JSON document out:

```bash
./query.sh query "SELECT COUNT(*) FROM orders WHERE status = 'stuck'"
./query.sh query --max-rows 5000 "SELECT * FROM events WHERE region = 'NE'"
./query.sh query --conn warehouse "SELECT COUNT(*) FROM inventory"
./query.sh query-file report.sql
./query.sh query --format tsv "SELECT id, name FROM customers"   # cheaper on tokens
```

The output contract is built for a consumer that reasons about what it got back: a row cap (default 1000) with an explicit `truncated` flag instead of silent truncation, the effective `maxRows` echoed back, `durationMs`, a `moreResultSets` signal for SQL Server batches, and errors as `{"error":"..."}` on stderr with a non-zero exit — connection string scrubbed.

Because the pod is cheap (50m CPU / 64Mi requests), the README's suggested shape is one pod per developer. But a team *may* share a single read-only pod — a cheap way to hand query access to a whole fleet of agents at once.

## Where the Trust Boundary Actually Is

I want to be precise about the limit, because it's the design's one sharp edge: **RBAC controls who can exec into a pod, not which flags they pass.** Anyone who can exec can use `--write` if that pod has a write key. So one pod equals one set of credentials — never set a write key on a shared pod. Writes go through a personal pod.

Beyond that, the current limitations are the boring kind: raw SQL only (no bound parameters yet), write mode reports `rowsAffected` without returning rows, and MySQL `DECIMAL` values may serialize as JSON strings — the driver's lossless representation.

## Try It

A prebuilt multi-arch image is published on every release, so trying it is a secret and a deploy:

```bash
export NAMESPACE=dev-alice
./manage.sh setup-namespace
./manage.sh set-conn "postgres://reader:pass@pg:5432/mydb?sslmode=require"
REGISTRY=ghcr.io/nathanfox IMAGE_TAG=v0.6.0 ./manage.sh deploy

./query.sh query "SELECT 1"
```

The repo also ships per-engine example secret manifests, an integration test that exercises the whole flow against a throwaway in-cluster Postgres, and a design doc recording the decisions — including the rejected alternatives.

---

**Source:** [GitHub (MIT)](https://github.com/nathanfox/sqlpod) | **Image:** `ghcr.io/nathanfox/sqlpod`
