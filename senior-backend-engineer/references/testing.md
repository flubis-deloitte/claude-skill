# Backend Testing Playbook

How a senior tests Go services: fast, deterministic, and proving behavior that matters.
Load this when writing or reviewing tests.

## The shape of a good test suite

- **Unit tests** — business/domain logic in isolation. Fast, no I/O. The bulk of tests.
- **Integration tests** — the data layer and handlers against real dependencies
  (containerized Postgres/MySQL/Mongo/Redis). Fewer, but they catch the bugs unit tests
  can't (SQL, transactions, serialization).
- **End-to-end** — a thin layer over the running service for critical happy paths only.

Aim coverage at behavior and edge cases, not a percentage number. Test the unhappy
paths (errors, timeouts, conflicts) — that's where bugs live.

## Table-driven tests (the Go default)

```go
func TestDiscount(t *testing.T) {
    tests := []struct {
        name    string
        in      Order
        want    int
        wantErr error
    }{
        {"no items", Order{}, 0, ErrEmptyOrder},
        {"single item", Order{Items: []Item{{Price: 100}}}, 100, nil},
        // ...
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Discount(tt.in)
            if !errors.Is(err, tt.wantErr) {
                t.Fatalf("err = %v, want %v", err, tt.wantErr)
            }
            if got != tt.want {
                t.Errorf("got %d, want %d", got, tt.want)
            }
        })
    }
}
```

- Subtests with `t.Run` for clear failure names. Use `t.Parallel()` where safe.
- Prefer the standard library; `testify` for asserts is fine if the repo uses it. Don't
  introduce a new assertion framework against repo convention.

## Testing handlers

- Use `net/http/httptest`:

```go
req := httptest.NewRequest(http.MethodGet, "/users/42", nil)
rec := httptest.NewRecorder()
router.ServeHTTP(rec, req)
if rec.Code != http.StatusOK { t.Fatalf("status = %d", rec.Code) }
```

- Assert status code, body shape, and headers. Cover validation failures and error
  mappings, not just the happy path.

## Testing the data layer

- Prefer **real databases** over mocks for repositories — use `testcontainers-go` or a
  disposable test DB. Mocked SQL hides the bugs that matter (syntax, constraints, tx
  behavior).
- Run migrations against the test DB; seed minimal fixtures per test; clean up (truncate
  or transaction-rollback per test) for isolation.
- For Redis, run a real instance/container; test TTLs, eviction, and lock behavior.

## Mocking — sparingly and at boundaries

- Mock external systems you don't own (third-party HTTP APIs) via `httptest.Server` or
  an interface fake. Define the interface in the consumer and inject a fake in tests.
- Don't mock your own database driver. Don't mock so much that the test only proves the
  mock was called.

## Determinism

- Inject `time.Now` (a clock), random sources, and ID generators so tests are
  reproducible. No `time.Sleep` to "wait" for concurrency — synchronize with channels or
  `sync.WaitGroup`.
- Always run `go test ./... -race`. Concurrency bugs that pass without `-race` are still
  bugs.

## What to assert

- Behavior and contracts, not implementation details. A refactor that preserves behavior
  shouldn't break tests.
- Error cases: wrong input → right error/status; downstream failure → graceful handling;
  retries/idempotency where relevant.

## Anti-patterns

- Mock-heavy "unit" tests of repositories that never touch SQL.
- Flaky tests using sleeps and wall-clock time.
- Asserting on log output or private fields instead of observable behavior.
- One giant test instead of named table cases.
- Skipping the `-race` flag.
- Chasing a coverage % with trivial getters while error paths go untested.
