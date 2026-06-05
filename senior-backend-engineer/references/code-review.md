# Backend Code Review Playbook

How to review Go/backend changes like a senior. Be specific, kind, and prioritized —
distinguish blockers from nits. Quote the line, explain the risk, suggest the fix.

## Order of review

1. **Understand the intent.** Read the PR description and the contract being changed.
   If you can't tell what "done" means, that's the first comment.
2. **Correctness & data integrity first**, then concurrency/security, then performance,
   then style. Don't bikeshed naming while a transaction bug sits unreviewed.

## Correctness

- Are all errors checked, wrapped with context (`%w`), and mapped to the right status
  code at the boundary? Any `_ = err` that hides a real failure?
- Are unhappy paths handled: not-found, conflict, validation, timeout, downstream failure?
- Off-by-one, nil dereference, unchecked type assertions (`v.(T)` without the `, ok`)?
- Does it do what the PR claims, with tests that actually prove it?

## Concurrency

- Any shared state mutated without synchronization? Would `go test -race` flag it?
- Goroutines with no exit path or not tied to a context (leaks)? Unbounded fan-out?
- Correct use of channels/`WaitGroup`/`errgroup`? Deadlock or lost-signal risk?

## Context & resources

- Is `context.Context` threaded through and respected? Any `context.Background()` in a
  request path? Missing timeouts on outbound calls?
- `defer` close on rows, bodies, files? Pooled clients reused, not created per request?

## Data layer

- Parameterized queries everywhere (no string-built SQL)?
- N+1 queries that should be a join/batch? `SELECT *` pulling more than needed?
- Transaction scope correct and short? Right isolation level? Idempotent where retried?
- Migration included, reversible, and safe to run on a large table without long locks?
- Indexes for new query patterns? Money as decimal, timestamps UTC?

## Security

- Input validated and authZ enforced before work happens?
- Any secrets/tokens/PII logged or returned to the client?
- Injection (SQL/command/template), SSRF on outbound URLs, path traversal on file ops?
- Least-privilege assumptions intact?

## API contract

- Status codes accurate; error envelope consistent; no DB models serialized directly?
- Backward compatible within the version (no removed/renamed/repurposed fields)?
- Pagination/limits on new list endpoints?

## Performance

- Hot path allocations, unnecessary copies, missing batching?
- Caching used where it helps — with a TTL and invalidation story?
- Any operation that's O(n) DB round-trips when it could be O(1)?

## Tests & DX

- New logic covered by table-driven unit tests; handlers by `httptest`; data layer by
  integration tests against a real/containerized DB?
- `go build`, `go vet`, linter, and `go test -race` green?
- Dead code, stray prints, commented-out blocks removed?

## How to phrase it

- **Blocker:** "This builds SQL with `fmt.Sprintf` — injectable. Use a parameterized
  query: `db.QueryContext(ctx, "... WHERE id = $1", id)`."
- **Suggestion:** "Consider batching these into one `WHERE id = ANY($1)` to avoid the
  N+1."
- **Nit (non-blocking):** "nit: `userSvc` reads cleaner than `us`."

Approve when correctness, safety, and tests are solid — don't hold a PR hostage over
preferences. Be explicit about what's blocking vs. optional.
