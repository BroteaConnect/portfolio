# Responsive and mobile layout

Until PR #3 this page had no working mobile layout at all — not because
media queries were wrong, but because they could never apply. The entire
layout lived in inline `style` attributes, which beat any stylesheet rule:
fixed 2/3/4-column grids that never collapsed, fixed 46–56px headings,
fixed image heights (300–460px), and a non-wrapping header row that
overflowed on every phone. The wrapper's `overflow-x:hidden` then *hid*
the damage instead of preventing it — the cut-off nav links and CTA were
simply unreachable.

The fix: every layout-critical inline property was extracted to classes
in the page's `<style is:global>` block (`src/pages/[...lang]/index.astro`).
Colors and decoration stay inline, on theme tokens, as before. Desktop at
1280px is unchanged.

## Breakpoints

All in the page's global style block, all `max-width`:

| Breakpoint | What changes |
|---|---|
| **1024px** | Dense grids drop to 2 columns (services/products `pf-grid-3`, team, stats); header gaps tighten. |
| **950px** | Inline nav row is replaced by the mobile menu (button + dropdown panel); CTA and nav/switcher links get ≥44px targets. |
| **720px** | Everything goes single-column: hero, section heads, services/products, projects, partners, contact and the form's name/org row. Footer links get 44px targets. |
| **600px** | The language switcher leaves the header row and moves into the menu panel (there is no room for logo + switcher + CTA + button at 320px). |
| **480px** | Spacing trims: tighter header gap, smaller CTA padding, the CTA's `→` arrow is hidden. |

**Stats and team stay 2-up on phones** — they never collapse to one column.

## Mobile navigation

Below 950px a 44×44px toggle button (`#pf-menu-btn`, animated
hamburger/X icon) opens a full-width dropdown panel under the sticky
header. Links in the panel are 48px tall. The panel closes on link click
and on `Escape` (which returns focus to the button, since the panel it
was in becomes `display:none`).

The button's accessible labels are **server-rendered** from the two
locale keys added in this feature, `navMenuOpen` / `navMenuClose`
(present in both `src/locales/en.json` and `src/locales/es.json`):

```astro
<button
  id="pf-menu-btn" class="pf-menu-btn" type="button"
  aria-expanded="false" aria-controls="pf-nav"
  aria-label={t(locale, 'navMenuOpen')}
  data-label-open={t(locale, 'navMenuOpen')}
  data-label-close={t(locale, 'navMenuClose')}
>
```

This pattern is not optional: Astro serves **one JS bundle for both
locale routes** (`/` and `/en/`), so any text hardcoded in the script
would ship in a single language. Labels travel as server-rendered
`data-*` attributes and the script only swaps `aria-label` between them
— the same pattern the contact form already uses for its status
messages.

Below 600px the panel also contains a second `LanguageSwitcher`
instance (`.pf-nav-lang`); the header instance is hidden.

Anchors opened from the menu land correctly because of
`scroll-padding-top: 88px` on `html` — without it the sticky header
covers the section title.

## Fluid type and images

Fixed pixel sizes became `clamp()`; the desktop (1280px) end of each
range equals the old fixed value, so desktop is pixel-identical:

- `h2`: `clamp(30px,6vw,46px)`; contact `h2`: `clamp(34px,7vw,56px)`;
  stat numbers: `clamp(34px,9vw,52px)`; hero `h1` floor lowered from
  44px to 36px (`clamp(36px,5.4vw,76px)`).
- Images are sized by `aspect-ratio` + `max-height` instead of fixed
  heights: hero `1/1` (max 460px), project cards `16/10` (max 320px),
  team portraits `4/5` (max 300px), all keeping `object-fit:cover`.
- Section padding, gaps and radii are fluid too, e.g.
  `padding: clamp(48px,10vw,96px) clamp(16px,4vw,32px)`.

## Touch targets

Nav links (48px), footer links, the CTA, the menu button and the
language-switcher links all meet the 44px minimum on mobile. The
switcher is a factory-generated component
(`src/components/LanguageSwitcher.astro`) and **must not be edited** —
it is styled from the page CSS via `.lang a { … }`. Inside the menu
panel that requires `!important` on a couple of properties, because the
component's scoped CSS (`data-astro-cid` attribute selectors) beats
page-level rules in specificity/order.

## Accessibility and robustness

- Sticky header: `-webkit-backdrop-filter` alongside `backdrop-filter`;
  where neither is supported, the existing `rgba(9,9,45,.72)` background
  reads as a solid-enough fallback.
- The marquee animation runs only under
  `@media (prefers-reduced-motion: no-preference)`.

## Rules when editing `src/pages/[...lang]/index.astro`

1. **Never put layout-critical properties back into inline `style`
   attributes** — `display`, `grid-template-columns`, `flex`, `gap`,
   `padding`, `font-size`, `height`. Inline styles override the media
   queries; one reintroduced property silently breaks mobile while
   desktop keeps looking fine. Layout belongs to the `pf-*` classes in
   the global style block, and only there.
2. **Test both locales at 320px.** Spanish copy is longer than English;
   a layout tuned on `/en/` can clip on `/`.
3. **New user-facing copy always goes through locale keys** in both
   `src/locales/en.json` and `src/locales/es.json` — including
   `aria-label`s. CI (`src/locales/locales.test.mjs`) blocks a
   one-language key. If the text is needed in JS, server-render it into
   a `data-*` attribute; never write literals in the script.
