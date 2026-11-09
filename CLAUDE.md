# CLAUDE.md

## What this is

`@sudobility/genui` — a React library that renders a server-driven UI tree (an
`IRenderable`, Kotlin/iOS-style payload) into React components. Render-only; all
interaction is reported via one callback `onAction(value, renderable)`. Published
as a **restricted** package to **GitHub Packages**, not npmjs.

## Stack

- React 19, TypeScript 6, Vite 8 (library mode), Vitest 4, ESLint 10.
- Package manager: **Bun** (`bun@1.3.10`). Use `bun run <script>`, not npm/yarn.
- ESM-only output (`"type": "module"`, single `es` build).
- Peer deps (not bundled): `@sudobility/components`, `@sudobility/design`,
  `@sudobility/genui_types`, `@vis.gl/react-google-maps`, `swiper`, `react`,
  `react-dom`. Shared types live in `@sudobility/genui_types`; `src/types.ts`
  only re-exports them plus `GenUIProps`.

## Commands (all verified)

- `bun run build` — `tsc` + `vite build` + `node scripts/fix-esm-imports.mjs`.
- `bun run type-check` — `tsc --noEmit`.
- `bun run lint` — eslint, `--max-warnings 0` (zero warnings or it fails).
- `bun run format` / `format:check` — prettier.
- `bun run test:run` (alias `test:unit`) — `vitest run`. Currently 5 tests.
- `bun run dev` — `vite build --watch`.

## Architecture

- `src/index.ts` — public entry; exports `GenUI` + types/enums.
- `src/gen-ui.tsx` — `<GenUI>` wrapper; provides `GoogleMapsApiKeyContext`.
- `src/render-node.tsx` — **the core**. `RenderNode` dispatches on `view.layout`
  (a long if/Set chain) to per-layout render functions. Unknown layout →
  `renderBasicCard` fallback. This is where you add/modify rendered layouts.
- `src/theme.ts` — maps payload `ThemeColor`/`FontWeight` tokens → Tailwind
  classes. Semantic roles → `@sudobility/design` tokens (`text-foreground`,
  `bg-muted`, …); `SYSTEM_*`/`WHITE`/`BLACK`/`*_GRAY` → fixed Tailwind palette
  (intentional, per payload's iOS UIColor semantics — don't "fix" these).
- `src/utils.ts` — payload helpers (`labelText`, `imageSrc`, `actionValueOf`,
  `viewOf`, input-layout predicates).

## Key conventions / gotchas

- The host app must have Tailwind + the `@sudobility/design` token layer; GenUI
  emits utility classes, it ships no CSS (except inline critical Swiper CSS,
  injected once at runtime via `ensureSwiperCss`).
- Map layouts (`map`, `map_pin`) render only when `googleMapsApiKey` is passed;
  otherwise they show a fallback message.
- Inputs/toggles/sliders/selects fire `onAction` with their initial value on
  mount, then on each change.
- `RenderNode` defensively merges `renderable.children` into `view.children`
  when the AI-generated payload misplaces them (see top of `RenderNode`).
- Build uses `moduleResolution: "bundler"` (extensionless imports), so
  `scripts/fix-esm-imports.mjs` rewrites emitted relative imports to explicit
  `.js` so Node's native ESM loader resolves them. Build output is not valid
  for Node without it — keep that script in the `build` script.
- `.npmrc` pins `@sudobility` scope to `https://npm.pkg.github.com`; installing
  needs a GitHub token with `read:packages`.
- CI (`.github/workflows/ci-cd.yml`) calls the shared `johnqh/workflows`
  pipeline with `publish-target: github`; auto-publishes on push to
  `main`/`develop`. `prepublishOnly` runs the full build.
- Raw color literals exist in Swiper critical CSS and SVG map-pin markers;
  comments mark them as decorative third-party/graphic colors, not themable
  surfaces — leave them.
