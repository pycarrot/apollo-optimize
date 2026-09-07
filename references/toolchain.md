# Toolchain catalogue

Load when the change touches build config, CI, test setup, linting, package management,
or the developer loop. A slow loop taxes every future change; optimizing it compounds.

## Don't do it

- Run only affected tests locally (`vitest --changed`, project filters); run the full suite
  in CI and before commit.
- No dependency for something the platform, the language, or fifty lines of code can do.
  Each dependency costs install time, bundle bytes, audit surface, and upgrade churn.
- Remove unused scripts, config keys, and CI steps. Read the workflow file: every step
  that runs on every push should justify its minutes.
- Do not build what did not change: incremental TypeScript (`tsc -b`), cached bundler
  output, Turborepo/Nx task caching keyed on inputs.

## Do it once

- Cache in CI: package store, build outputs, test caches, browser binaries. Key caches on
  lockfile hash; restore before install.
- Snapshot expensive fixtures (compiled content, seeded databases) once per run, not per
  test.
- Compile schemas / content at build time, ship the compiled form, validate once in CI.

## Do it later, in bulk

- Split CI into fast (lint, typecheck, unit) and slow (integration, e2e) lanes; gate merge
  on fast, run slow in parallel or on a schedule.
- Batch lint fixes with the tool (`biome check --write`), never by hand.

## Better algorithm

- Test isolation with in-memory or per-worker databases instead of shared state that
  forces serialization.
- Typecheck with project references so packages check independently and cache.
- Parallel test workers sized to cores; isolate only the tests that need it.

## Smaller

- Lean lockfile: one version of each package where possible; dedupe (`pnpm dedupe`).
- `sideEffects: false` in package manifests so bundlers can tree-shake.
- Ship source maps to an error tracker, not to users, unless debugging in the field is
  required.
- Keep CI images small; install only the toolchain the job uses.

## Parallel

- Matrix jobs for independent packages; fail fast.
- Local: `pnpm -r --parallel` for independent builds; watch modes instead of rebuilds.

## Tighter

- Fast linters (Biome, oxlint) over slow ones; single pass for lint + format.
- esbuild/SWC for transforms; reserve tsc for type checking only.
- Pre-commit hooks that run in under two seconds on the staged files, or none at all.

## Measure

- `time` every command in the loop: install, build, typecheck, test, lint. Write the
  numbers down; re-measure after changes.
- CI: total duration and per-step duration over the last ten runs; the longest step is the
  target.
- Test-suite duration in CI output; a growing suite should not grow linearly in time.
