# Backend and edge catalogue

Load when the change touches servers, serverless / edge runtimes, APIs, middleware,
caching layers, auth, or scheduled jobs.

## Don't do it

- Reject early. Validate and authorize before any I/O; a bad request should cost
  microseconds and zero database time.
- Do not compute what the client can render. Send data, not formatted strings; send ids,
  not joined blobs the client already has.
- Remove per-request work that has the same answer every time: config parsing, schema
  compilation, regex construction, key derivation. Do it once at module scope.
- No logging in hot paths beyond what someone will actually read. Structured, sampled,
  and level-gated.
- Do not retry non-idempotent operations. Do not retry at all without backoff and a cap.

## Do it once

- Module-scope caches for anything immutable during the deployment lifetime: compiled
  validators, static lookup tables, public keys.
- Cache at the edge (`Cache-Control`, `ETag`, `stale-while-revalidate`) for anything
  public and anything per-user that changes rarely; put the user id in the cache key.
- Derive on read rather than write on tick. If a value can be computed from timestamps
  and a few stored numbers (streaks, regen timers, expiry), compute it in the handler and
  never store it.
- Memoize expensive crypto (key derivation, JWKS fetch) with a TTL.

## Do it later, in bulk

- Batch writes: collect events, write one row per batch or one statement with many rows.
  Flush on a timer or a threshold.
- Move aggregation to a cron / queue consumer. User requests should not compute
  statistics; they should read precomputed ones.
- Use queues for anything the user does not need to wait for: email, webhooks, analytics,
  cache warming.
- Respond, then work: `waitUntil` / background tasks for post-response bookkeeping.

## Better algorithm

- One round trip. Combine dependent queries into one statement (CTE, join, batch API);
  combine independent ones into a single batch call.
- Choose the storage by access pattern: KV for hot single-key reads, relational for
  queries, object storage for blobs, in-memory for per-isolate scratch.
- Avoid N+1 at the handler level: load the set, then map, never map-then-load.
- Pagination by cursor, not offset; offset pagination scans everything it skips.

## Smaller

- Trim response bodies: only fields the client reads, short keys where payloads are large
  and numerous, numbers not numeric strings, omit nulls.
- Stream large responses; never buffer the whole thing to compute a length.
- Keep the deployed bundle small: cold start is proportional to bytes parsed. Audit
  dependencies; edge runtimes punish heavy SDKs.
- Minimal middleware chain; each layer runs on every request.

## Parallel

- `Promise.all` independent I/O inside a handler; never sequential awaits for independent
  work.
- Fan out to subrequests only after minimizing each one; parallelism multiplies cost.

## Tighter

- Avoid `JSON.parse`/`stringify` round trips between layers that could pass objects.
- Reuse `TextEncoder`, `URL`, regex instances.
- Pre-size arrays and avoid intermediate arrays in transform chains on hot paths.
- Choose `Response` construction with a stream or a string, not both.

## Platform limits to know (verify against current docs)

- Cloudflare Workers: CPU time per request (10 ms free / 30 s paid configurable),
  128 MB memory, 1 MB script size (free) / 10 MB (paid), subrequest caps, D1 row
  read/write daily quotas on free tier, KV eventual consistency (~60 s), cache API
  is per-colo.
- AWS Lambda: cold start scales with package size and runtime init; memory sets CPU.
- Node servers: event-loop blocking >10 ms hurts every concurrent request; offload CPU
  work to workers.

## Measure

- Per-request CPU time and wall time from the platform (headers, logs, `wrangler tail`).
- Count subrequests and DB statements per handler; assert the count in a test where the
  platform allows it.
- Load test with a realistic mix before and after; p95 matters more than mean.
