# Go Service & Handler Playbook

How to build and change Go HTTP services the way a senior would. Load this for any
handler, middleware, service, or general Go work.

## Layering

Keep three clear layers and don't let them leak into each other:

- **Transport (HTTP)** — routers, handlers, middleware. Parses/validates input,
  calls the service, maps results and errors to HTTP. Knows about `http.Request`,
  status codes, JSON. Knows nothing about SQL.
- **Service / domain** — business logic and orchestration. Pure-ish, testable. Knows
  about domain types and repository interfaces. Knows nothing about `http` or SQL drivers.
- **Repository / data** — DB and external-system access. Implements interfaces the
  service defines. Knows about SQL/Mongo/Redis. Returns domain types or typed errors.

Define interfaces **where they are consumed** (in the service package), not where
they're implemented. Keep them small.

## Context propagation

- Every exported method that does I/O takes `ctx context.Context` as the first arg.
- Derive request scope from `r.Context()`; never `context.Background()` in a handler.
- Set deadlines on outbound work:

```go
ctx, cancel := context.WithTimeout(r.Context(), 3*time.Second)
defer cancel()
user, err := h.svc.GetUser(ctx, id)
```

- Pass `ctx` to every DB call (`QueryContext`, `db.WithContext`, etc.) so cancellation
  actually frees connections.

## Errors

- Define sentinel/typed errors in the domain layer:

```go
var ErrNotFound = errors.New("not found")
var ErrConflict = errors.New("conflict")
```

- Wrap with `%w` as errors cross layers; never discard the chain:

```go
if err != nil {
    return fmt.Errorf("get user %s: %w", id, err)
}
```

- Map to HTTP **once**, at the transport boundary:

```go
switch {
case errors.Is(err, domain.ErrNotFound):
    writeError(w, http.StatusNotFound, "user not found")
case errors.Is(err, domain.ErrConflict):
    writeError(w, http.StatusConflict, "already exists")
default:
    log.Error("get user", "err", err) // log internal detail here, not to the client
    writeError(w, http.StatusInternalServerError, "internal error")
}
```

- Log an error at exactly one place (usually the boundary). Don't log-and-return up
  the stack — that produces duplicate noise.

## Handler shape

```go
func (h *UserHandler) Get(w http.ResponseWriter, r *http.Request) {
    id := chi.URLParam(r, "id")
    if id == "" {
        writeError(w, http.StatusBadRequest, "id required")
        return
    }
    user, err := h.svc.GetUser(r.Context(), id)
    if err != nil {
        h.mapError(w, err)
        return
    }
    writeJSON(w, http.StatusOK, toUserResponse(user))
}
```

- Decode with a size limit: `r.Body = http.MaxBytesReader(w, r.Body, 1<<20)`.
- Use a `json.Decoder` with `DisallowUnknownFields()` for strict request bodies.
- Always `defer r.Body.Close()` where you read it (and close response bodies on the
  client side).
- Never write to `w` after you've already written a status — return early.

## Concurrency

- `go test -race` must be clean. Guard shared maps/slices with a mutex or channel.
- Every goroutine must have a lifecycle tied to a context and a way to stop:

```go
g, ctx := errgroup.WithContext(ctx)
for _, item := range items {
    item := item
    g.Go(func() error { return process(ctx, item) })
}
if err := g.Wait(); err != nil { return err }
```

- Bound fan-out with a semaphore or worker pool; don't launch one goroutine per row
  of unbounded input.
- Use `sync.Once` for one-time init, `sync.Pool` only when profiling justifies it.

## Resource hygiene

- `defer rows.Close()` immediately after a successful query; check `rows.Err()` after
  the loop.
- Reuse a single `*sql.DB` / `*pgxpool.Pool` / `*redis.Client` / `*http.Client` for the
  process lifetime. Configure `SetMaxOpenConns`, `SetMaxIdleConns`, `SetConnMaxLifetime`.
- Give `http.Client` a `Timeout`; never use `http.DefaultClient` for outbound calls in prod.

## Config & startup

- Read config from env at startup; fail fast on missing required values.
- Construct dependencies (DB pool, clients, services, handlers) in `main`/`cmd` and
  inject them — no global singletons, no `init()` side effects for I/O.
- Implement graceful shutdown: catch `SIGINT/SIGTERM`, call `srv.Shutdown(ctx)`, close
  the DB pool.

## Anti-patterns

- Business logic inside handlers; SQL inside handlers.
- `panic` for recoverable errors in a request path.
- Returning naked `err` to the client (leaks internals) or swallowing it.
- Per-request client construction; missing timeouts; unclosed rows/bodies.
- Global mutable state shared across requests without synchronization.
