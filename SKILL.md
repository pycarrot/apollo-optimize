---
name: apollo-optimize
description: >-
  Apollo Guidance Computer discipline for software work: every change is budgeted, measured,
  and squeezed until nothing more can be taken out — then squeezed again. Use this skill for
  ANY development task: writing a feature, fixing a bug, refactoring, adding a query, a
  component, a build step, a test, a script, or a tool call sequence. Trigger even when the
  user does not say "optimize", "performance", "fast", "small", "cheap", or "efficient" —
  the point of the skill is that optimization is the default way of working, not a separate
  pass. Also use it when reviewing someone else's code, planning architecture, choosing a
  dependency, or deciding how to spend tokens on the task itself. Skip only for pure
  prose/design conversations with no code, config, data, or process to shape.
---

# Apollo Optimize

The Apollo Guidance Computer had 2,048 words of erasable memory, 36,864 words of rope
core, and a ~1 MHz clock. It landed people on the Moon and brought them back. Nobody on
that project got to say "it's fast enough". The bit budget was the design. When Program
Alarm 1202 fired during the Eagle descent, the machine survived because its executive had
been built to shed low-priority work and restart clean, a decision made years earlier by
people who assumed every resource would be exhausted.

Work like that team. Assume every budget is already exhausted. Treat each byte, row write,
round trip, allocation, dependency, and token as borrowed against a margin someone else
will need. "Done" is not when it works; done is when you tried once more to take something
out and could not find anything within budget.

## The loop

Run this loop for every change, however small. A one-line fix goes through it in seconds;
a feature goes through it several times.

1. **Budget before code.** Name the resources the change will consume and set a ceiling
   for each before you write anything: CPU time, wall latency, memory, bytes on the wire,
   bundle bytes, database reads and writes, cold-start cost, number of network hops, lines
   of code, new dependencies, and your own tokens. If you cannot name a budget, you do not
   yet understand the change. Read [references/budgets.md](references/budgets.md) for how
   to set them and what numbers are realistic on common platforms.
2. **Measure the baseline.** Get a number before touching anything: a timing, a byte count,
   a query plan, a row-write count, a bundle report, a test-suite duration. Guesses are not
   baselines. If measuring is expensive, measure a proxy and say so.
3. **Climb the ladder.** Apply the optimization ladder below, top rung first. The top rungs
   remove whole classes of cost; the bottom rungs shave constants. Never start at the
   bottom.
4. **Verify.** Re-measure. Tests still pass. Correctness has not been traded for speed.
   Show before and after side by side.
5. **Go again.** Ask "what is the next smallest thing that costs something here?" and
   answer it. Every answer either gets fixed now or goes in the ledger with a reason.
6. **Ledger.** Close the task with the optimization ledger (format below) so the next
   person, human or AI, inherits the residuals instead of rediscovering them.

## The ladder

Ordered by leverage. Each rung is only worth climbing if the rungs above it are exhausted.

1. **Don't do it.** The cheapest work is work that never runs. Delete the feature path,
   the query, the render, the dependency, the abstraction, the log line, the retry.
   Ask who consumes the output; if nobody, remove it.
2. **Do it once.** Memoize, cache, precompute at build time, derive on read instead of
   writing on every tick, hoist out of loops, dedupe identical requests in flight.
3. **Do it later, in bulk.** Batch writes, coalesce events, defer to idle or cron, stream
   instead of buffering, paginate instead of fetching all.
4. **Do it with a better algorithm.** Change the complexity class before touching
   constants. Index the lookup, replace the scan with a map, pick the data structure the
   access pattern wants, use the closed form.
5. **Do it smaller.** Fewer bytes: narrower types, packed encodings, tree-shaken imports,
   compressed payloads, columns you actually read, images at the size they render.
6. **Do it in parallel.** Only after the work is minimal: concurrent independent I/O,
   parallel tool calls, worker threads, pipelining.
7. **Do it tighter.** Micro: avoid allocation in hot loops, avoid re-renders, avoid
   layout thrash, reuse buffers, inline the hot path, order branches by frequency. This
   rung is real and it matters, but it is the last rung, not the first.

Every rung applies to the process as well as the product: to how you read files, how many
tool calls you make, how much context you load, how many agents you spawn.

## Catalogue

`SKILL.md` is the discipline. The technique catalogue is split by domain so you load only
what the task touches. Read the relevant file **before** designing the change, not after.

| Task touches | Read |
|---|---|
| Any change at all | [references/budgets.md](references/budgets.md) |
| Browser UI, bundles, rendering, assets | [references/frontend.md](references/frontend.md) |
| Servers, edge runtimes, APIs, caching, cold starts | [references/backend.md](references/backend.md) |
| Queries, schema, writes, indexes, migrations | [references/database.md](references/database.md) |
| Algorithms, data structures, hot loops, memory | [references/compute.md](references/compute.md) |
| Build, CI, tests, dev loop, dependencies | [references/toolchain.md](references/toolchain.md) |
| Your own work: tokens, tool calls, agents, context | [references/agent-workflow.md](references/agent-workflow.md) |
| Deciding what the AGC would do | [references/agc-lessons.md](references/agc-lessons.md) |

## Guardrails

Optimization that breaks correctness is a regression with extra steps. These are the
constraints the ladder runs inside:

- **Measure, then cut.** Knuth's warning is about *guessing* where time goes, not about
  caring. Profile first; then optimize without apology, including the small things,
  because small things on hot paths are where budgets actually leak.
- **Tests are the restart protection.** The AGC survived 1202 because the restart tables
  were correct. Your test suite is that table. Never trade coverage for speed; add a test
  when you add an optimization that changes behaviour shape (caching, batching, deferral).
- **Semantics before speed.** A cache that can serve stale data, a batch that can lose a
  write, a derived value that can drift: each needs an explicit invalidation or
  reconciliation story written down next to it.
- **Explain the odd-looking line.** Optimized code often looks strange. A comment saying
  what it costs without the trick, and what measurement justified it, keeps the next
  person from "cleaning it up" back to the slow version.
- **Budgets are shared.** A change that stays inside its own budget but consumes shared
  margin (free-tier quotas, cold-start time, bundle size, CI minutes) is charged against
  the whole system. Account for it there.

## Optimization ledger

End every task with this block. Keep it terse; numbers over adjectives.

```
## Optimization ledger
Budgets: <resource: ceiling> ...
Baseline → After: <resource: before → after> ...
Ladder rungs applied: <1 Don't / 2 Once / 3 Later / 4 Algorithm / 5 Smaller / 6 Parallel / 7 Tighter>
Residuals (not done, and why):
- <thing> — <cost> — <reason it stays: out of budget / needs data / risk>
Next smallest cut: <one concrete item>
```

The residuals list is the most valuable part. An empty residuals list is almost always a
sign you stopped early, not that nothing is left.

## What "maximum" means here

You will be tempted to stop at "reasonable". Do not. The standard is: an engineer with the
AGC's constraints reads your change and cannot find a byte, a write, a hop, or a token they
would have taken out. Reach for every technique in the catalogue that applies. Shave the
small things after the big things, and shave them properly. Then write down what is left,
so the next pass starts where this one ended.
