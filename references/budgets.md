# Budgets

A budget is a ceiling you set before writing code, so that the code is shaped by the
constraint instead of the constraint being discovered in production. The AGC team knew
the word count of every routine before it was written. Do the same.

## How to set one

1. List every resource the change touches. Use the checklist below; most changes touch
   four or five.
2. For each, write the current number (baseline) and the ceiling you will accept.
3. If a ceiling is "unknown", measure or look up the platform limit before proceeding.
4. Keep the budget in the task notes or the PR description. It is the acceptance test.

## Resource checklist

| Resource | How to measure | Typical ceiling to reason from |
|---|---|---|
| CPU time per request | platform CPU-time header, `performance.now()` around the handler, profiler | Edge workers: 10–50 ms free tier. Node API: <100 ms p95 |
| Wall latency (p50 / p95) | request timing, RUM, synthetic probe | Interactive UI action <100 ms perceived; page nav <1 s |
| Memory (peak) | heap snapshot, `process.memoryUsage()`, `/usr/bin/time -l` | Edge: 128 MB. Lambda default 128 MB. Browser tab: keep heap flat over a session |
| Bytes on the wire per request | DevTools network, `content-length`, `curl -w '%{size_download}'` | API JSON <10 KB typical; anything >100 KB needs pagination or streaming |
| Bundle size (gzip / brotli) | bundler report, `size-limit`, `source-map-explorer` | Initial JS <100 KB gz for an app; <50 KB for a landing page |
| Database reads / writes per operation | query log, driver counters, `EXPLAIN` | Free tiers cap *writes per day*; treat one row write as expensive |
| Network hops per user action | trace, waterfall | 1 for reads, ≤2 for writes. Sequential hops multiply latency |
| Cold start | first-request timing after deploy / idle | Edge: <5 ms. Lambda/Node: <300 ms. Bundle bytes drive this |
| Dependencies added | lockfile diff, `npm ls --depth=0` | Zero unless it saves >200 lines of correct, tested code |
| Lines of code | diff stat | Smaller diffs are cheaper to review, ship, and revert |
| Test-suite duration | `time pnpm test` | Additions should be sub-second unless they're integration tests |
| CI minutes | workflow duration | Cache, split, and skip; CI is a shared budget |
| Your tokens | context size, tool-call count | See `agent-workflow.md` |

## Cost at scale

Always multiply by the thing that grows. A write per user per day is trivial at 10 users
and a quota breach at 10,000. A 5 KB payload is fine once and 500 MB across 100k page
views. Write the multiplication out: `cost × users × frequency` and compare it with the
ceiling. If the product exceeds a shared quota (free tier, rate limit, monthly bill),
the change is out of budget regardless of its per-call cost.

## When the budget cannot be met

Say so, with the number, and propose which rung of the ladder would bring it back. Do not
quietly ship over budget; do not quietly scale down the feature. The budget decision
belongs to the person who owns the system.
