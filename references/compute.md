# Compute catalogue

Load when the change touches algorithms, data structures, hot loops, numeric work,
serialization, memory, or anything measured in microseconds.

## Don't do it

- Early exit. Check the cheapest disqualifying condition first; order predicates by
  (probability of rejecting) / (cost).
- Lazy evaluation: compute the expensive branch only when the result is consumed.
- Skip work that cannot change the answer: bounded searches, pruning, short-circuit on
  already-known results.

## Do it once

- Hoist invariants out of loops; precompute lookup tables for pure functions over small
  domains.
- Memoize pure functions with bounded, keyed caches (LRU). Unbounded memoization is a
  memory leak with a good excuse.
- Compile once: regexes, schemas, templates, format strings at module scope.

## Do it later, in bulk

- Amortize: grow buffers geometrically; process in chunks that fit cache; sort once then
  binary-search many times.

## Better algorithm

- Know the complexity of every loop you write and every library call inside it. A
  `.find` inside a `.map` is O(n²); build a `Map` first.
- Pick the structure for the access pattern: `Map`/`Set` for membership, arrays for
  ordered scans, heaps for top-k, tries for prefix, bitsets for dense boolean sets,
  sorted arrays with binary search for static lookups.
- Use closed forms and incremental updates where the math allows (running mean/variance,
  prefix sums, rolling windows) instead of recomputing from scratch.
- For scheduling / rating / probability models (FSRS, IRT, Elo-style systems), cache
  intermediate terms that do not depend on the candidate being evaluated; evaluate
  candidates against precomputed context.
- Integer arithmetic where floats are not required; fixed-point for money and scores.

## Smaller

- Typed arrays for numeric vectors; `Uint8Array` for bytes, not arrays of numbers.
- Compact encodings on the wire and in storage (bit flags, delta encoding, varints) when
  volume justifies it.
- Struct-of-arrays over array-of-structs when iterating one field across many items.
- Avoid retaining references that keep large graphs alive; clear caches on phase change.

## Parallel

- Only after the work is minimal and profiled: workers for CPU-bound tasks, chunked so
  the main thread stays responsive. Measure the transfer cost; copying large payloads to
  a worker can cost more than the work.

## Tighter

- No allocation in the hot loop: reuse arrays and objects, avoid spread/destructure in
  per-item code, avoid closures created per iteration.
- Monomorphic call sites: same shape of object through the same function; mixed shapes
  deoptimize.
- Avoid `try/catch`, `arguments`, and `delete` in hot code on older engines; check the
  profiler before assuming on modern ones.
- Prefer `for` loops over chained `.map().filter().reduce()` on hot paths (each stage
  allocates an intermediate array); keep the chain where it is not hot, for clarity.
- String building with arrays and one `join`, or template literals; never repeated `+=`
  in large loops.
- Branch by frequency; put the common case first.

## Measure

- Microbenchmarks with warm-up and many iterations (`vitest bench`, `tinybench`); compare
  medians, not single runs; run both versions in the same process.
- CPU profiler for where time goes; allocation profiler for where memory goes.
- Assert complexity with a scaling test: 10×, 100× input should scale as expected.
