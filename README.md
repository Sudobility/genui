# GenUI

`@sudobility/genui` is a React renderer for server-driven UI. It takes a single
`renderable` tree — a Kotlin/iOS-style payload shape shared via
[`@sudobility/genui_types`](https://github.com/Sudobility) — and renders it into
React components built on [`@sudobility/components`](https://github.com/Sudobility)
and [`@sudobility/design`](https://github.com/Sudobility).

The library only renders. It holds no routing or data-fetching logic; all user
interaction is reported back to the host app through a single callback:

```ts
onAction(value: string, renderable: IRenderable): void
```

## What it renders

`RenderNode` dispatches on `view.layout` (see `src/render-node.tsx`). Supported
layout families include:

- **Text / cards** — `paragraph`, `text_markup`, `footer`, `center`,
  `center_image`, the `line_*` family (`line_title`, `line_title_subtitle_value`,
  `line_image_title`, …), and a basic card fallback for unknown layouts.
- **Collections** — `list*`, `grid*`, `stacked*`, `spaced_*`, `tabs_*`,
  `carousel` (Swiper coverflow), `calendar`, `header`.
- **Actions** — `action`, `line_action`, `fab*`, `nav_item`, `tile`, `chip`
  (rendered as buttons that fire `onAction`).
- **Inputs** — `input_text`, `input_numeric`, `input_password`, `input_email`,
  `input_phone`, `input_date`, `input_text_block`, `search`, plus `line_toggle`,
  `line_slider`, and `line_select`.
- **Media / misc** — `just_image`, `embedded_image`, `web` (iframe), `link`,
  `waiting` (spinner), `spacer_horizontal` / `spacer_vertical`.
- **Maps** — `map` and `map_pin`, rendered with
  [`@vis.gl/react-google-maps`](https://www.npmjs.com/package/@vis.gl/react-google-maps)
  (requires `googleMapsApiKey`).

Visual styling (colors, font weights) comes from `ThemeColor` / `FontWeight`
tokens on the payload, mapped to `@sudobility/design` semantic Tailwind classes
in `src/theme.ts`.

## Install

This is a **restricted package published to GitHub Packages**. Consumers need an
`.npmrc` that points the `@sudobility` scope at the GitHub registry and
authenticates with a token that has `read:packages`:

```
@sudobility:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${GH_PACKAGES_READ_TOKEN}
```

Then install the package and its peer dependencies:

```bash
bun add @sudobility/genui \
  @sudobility/components @sudobility/design @sudobility/genui_types \
  @vis.gl/react-google-maps swiper react react-dom
```

Peer dependencies are **not** bundled — the host app must provide them. The host
must also have Tailwind CSS configured with the `@sudobility/design` token layer,
since GenUI emits Tailwind utility classes and semantic tokens (`text-foreground`,
`bg-muted`, `text-primary`, …).

## Usage

```tsx
import { GenUI, type IRenderable } from '@sudobility/genui';

const tree: IRenderable = {
  id: 'inbox',
  view: {
    layout: 'line_title_subtitle_value',
    title: { text: 'Inbox Zero' },
    subtitle: { text: 'Keep mail under control' },
    valueText: { text: '12' },
  },
};

<GenUI
  renderable={tree}
  onAction={(value, renderable) => console.log(value, renderable.id)}
  googleMapsApiKey={process.env.GOOGLE_MAPS_API_KEY} // optional, enables map layouts
  className="max-w-xl" // optional, merged onto the root wrapper
/>;
```

`GenUIProps`: `renderable` (required), `onAction?`, `className?`, `googleMapsApiKey?`.

Inputs, toggles, sliders, and selects emit their initial value through
`onAction` on mount, then again on every change.

## Development

Requires [Bun](https://bun.sh) (`bun@1.3.10`, see `packageManager`).

```bash
bun install          # install dependencies
bun run build        # tsc + vite lib build + ESM import-extension fixup
bun run dev          # vite build --watch
bun run type-check   # tsc --noEmit
bun run lint         # eslint (zero warnings allowed)
bun run format       # prettier --write
bun run test:run     # vitest run (also: test:unit)
```

`bun run build` compiles an ESM-only library to `dist/` (`index.js` +
`index.d.ts`), then runs `scripts/fix-esm-imports.mjs` to rewrite relative
imports with explicit `.js` extensions so Node's native ESM loader can resolve
them (the build uses `moduleResolution: "bundler"`, which omits extensions).

## Testing

Tests use Vitest + jsdom + Testing Library (`src/__tests__/`). The vitest config
enforces 80% coverage thresholds. Run `bun run test:run`.

## Layout

```
src/
  index.ts          # public entry: exports GenUI + types
  gen-ui.tsx        # <GenUI> wrapper + GoogleMapsApiKeyContext
  render-node.tsx   # RenderNode: the layout -> component dispatch (core logic)
  theme.ts          # ThemeColor / FontWeight token -> Tailwind class maps
  utils.ts          # payload helpers (labelText, imageSrc, actionValueOf, …)
  types.ts          # re-exports @sudobility/genui_types + GenUIProps
scripts/
  fix-esm-imports.mjs  # post-build ESM import-extension fixup
```

## Publishing

CI (`.github/workflows/ci-cd.yml`) delegates to the shared
`johnqh/workflows` unified pipeline and publishes to GitHub Packages
(`publish-target: github`) on pushes to `main`/`develop`. `prepublishOnly` runs
`bun run build`.

## License

BUSL-1.1
