# Frontend catalogue

Load when the change touches browser UI, bundles, rendering, assets, or the network
between browser and API. Apply the ladder in order; the first three sections are the top
rungs.

## Don't do it

- Render only what is on screen: virtualize long lists; lazy-mount tabs, modals, and
  below-the-fold sections; defer non-critical routes with dynamic `import()`.
- Remove polyfills for browsers you do not support. Check the target list before adding one.
- Delete unused CSS, components, feature flags, and A/B branches. A tree-shaker cannot
  remove code that is reachable but never triggered.
- Do not ship a library for one function. `date-fns`'s one formatter beats `moment`;
  `Intl` beats both. A 300-byte hand-written helper beats a 30 KB dependency.
- Do not fetch what is already in memory. Check the query cache / store before issuing a
  request; dedupe identical in-flight requests.

## Do it once

- Cache derived values with `useMemo` only where the derivation is measurable; hoist
  constants and static JSX out of render.
- Memoize expensive children with `memo` when their props are stable; if props are not
  stable, fix the props (stable callbacks, stable object identity) first.
- Precompute at build time: static routes, content indexes, image dimensions, sprite
  sheets, CSS variables. Anything the same for every user should not be computed per user.
- Persist query results across navigation (TanStack Query, SWR) with sane `staleTime`.
  A default `staleTime` of 0 refetches on every mount; set it deliberately.

## Do it later, in bulk

- Coalesce state updates; batch DOM writes; use `requestAnimationFrame` for visual updates,
  `requestIdleCallback` for analytics and prefetch.
- Debounce input-driven fetches; throttle scroll and resize handlers; use
  `IntersectionObserver` instead of scroll listeners.
- Prefetch on intent (hover, viewport proximity), not on page load.
- Stream HTML / suspense boundaries so the shell paints before data arrives.

## Better algorithm

- Keyed lists with stable keys; never index keys on reorderable data.
- Selectors that return primitives or stable references, so subscribers do not re-render.
- Move heavy computation (parsing, diffing, search indexing) to a Web Worker.
- Use CSS for what CSS can do: transitions, layout, sticky, `content-visibility`, container
  queries. JS-driven layout is the slow path.

## Smaller

- Measure the bundle with a real report (`vite-bundle-visualizer`, `source-map-explorer`,
  `size-limit`). Add a size budget check to CI so it cannot regress silently.
- Import from the leaf (`lodash-es/debounce`), not the barrel. Barrels defeat tree-shaking
  in surprising ways; check the report.
- Fonts: subset to the glyphs used (Thai + Latin subset is far smaller than the full
  face), `font-display: swap`, preload only the one face above the fold, WOFF2 only.
- Images: serve at rendered size, modern formats (AVIF/WebP with fallback), explicit
  `width`/`height` to avoid layout shift, `loading="lazy"` below the fold, `decoding="async"`.
- Icons: inline SVG sprite or a single icon component; never an icon font, never one
  request per icon.
- Strip dev-only code with `import.meta.env.DEV` guards so it is eliminated in production.
- Prefer platform APIs (`fetch`, `Intl`, `URL`, `structuredClone`, `crypto`) over
  packages that wrap them.
- Compress: brotli on the edge/CDN, long `Cache-Control` with hashed filenames, immutable
  assets.

## Parallel

- Preload critical resources (`<link rel="preload">`, `modulepreload`); avoid request
  waterfalls where a component fetches only after its parent's fetch resolves. Hoist
  fetches to the route.
- `Promise.all` independent requests; never `await` in a loop for independent work.

## Tighter

- Avoid layout thrash: batch reads, then writes; never read `offsetHeight` in a loop that
  also writes styles.
- Use `transform` and `opacity` for animation; they do not trigger layout or paint.
- Avoid creating closures and objects inside hot render paths when a profiler shows them.
- Keep the main thread free during input: long tasks >50 ms show up as jank. Split with
  `scheduler.yield()` / `setTimeout(0)` chunks.
- Passive event listeners for scroll and touch.

## Measure

- Lighthouse / Web Vitals (LCP, INP, CLS) in a fresh profile, throttled to a mid-range
  phone. Desktop numbers on a dev machine are not a baseline.
- React Profiler or the Performance panel for render counts and commit duration.
- Coverage panel to find unused JS/CSS on first load.
- Network panel with cache disabled for first-visit cost, enabled for repeat-visit cost.
