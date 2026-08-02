# Porting the design artifact

This site did not start as code. It started as a design artifact — a single
file that renders in Claude's design canvas, on its own runtime. Everything
below is what that runtime did for the artifact and what the browser needs
instead, plus the two places where the port is honestly not a perfect copy.

Read this before porting the next artifact: none of these are optional
polish, they are things that silently do nothing in a real browser.

## What had to be converted

| The artifact used | Why it cannot ship | What it became |
|---|---|---|
| `{{ t.key }}` bindings | no template engine runs in the browser | `{t(locale, 'key')}`, resolved at build time |
| `<image-slot src placeholder>` | a custom element only its runtime paints | `<img>` with `alt`, `loading="lazy"`, `object-fit` |
| `style-hover="…"` / `style-focus="…"` | invented attributes; no browser applies them | real CSS rules on `[data-hv]:hover` / `[data-fv]:focus-visible` |
| `sc-camel-on-click="{{ setES }}"` + `localStorage` | language as client state is invisible to crawlers | language as **routes** (`/`, `/en/`) with `hreflang` |
| `onsubmit="{{ onSubmit }}"` | the handler lived in the runtime | one `<script>` posting to the chassis endpoint |
| `required="{{ true }}"` | the string `"{{ true }}"` is not a boolean | plain `required` |
| `@font-face` pointing at asset ids | the ids were canvas assets, gone after export | the theme's `@font-face`, served from `public/fonts` |
| brand colours as hex | the theme is the vocabulary of colour | theme tokens (see below) |
| `data-screen-label="Hero"` | labels for the design canvas, not the page | removed |

Two traps in that list are worth naming, because both compile green and both
break something a user would notice:

- **`:focus-visible` is not decoration.** The artifact's focus styles were in
  `style-focus` attributes. Drop them without converting and keyboard users
  lose every focus ring on the page.
- **`alt` comes from `placeholder`.** It is the only text the artifact carried
  for its images. The three product logos deliberately keep `alt=""`: the
  `<h3>` next to each already says the name, and a duplicate `alt` gets read
  twice by a screen reader.

## Colour and type: the theme is the vocabulary

The page holds **no brand hex**. The `--pf-*` variables at the top of its
style block map the artifact's palette onto theme tokens — `--pf-ground` is
`var(--bg)`, `--pf-accent` is `var(--primary)`, `--pf-violet` is
`var(--accent)`, because the dark mode of `brotea@2` *is* this brand palette.

Three brand colours are not in the token vocabulary yet (Bloom Pink and the
two night grounds, Node Green and Deep Garnet). They live as local variables,
commented as such. **If a second app needs any of them, they go up into the
theme** — that is the whole point of the contract.

One bug worth remembering: the artifact declared its own `--font-display`,
which **shadowed the theme token of the same name**. The theme's is the Inktrap
cut; the artifact's was the plain one. Every heading on the page silently
rendered in the wrong typeface, through a green build and a green CI. It was
only visible by opening the deployed page in a real browser and reading
`getComputedStyle`. Never redeclare a token name the theme owns.

The artifact also carried its entire design system inline — neutral ramp, size
scale, shadows, gradients, easings: 68 variables the page never referenced.
That is the theme's vocabulary copied by hand into a place where it would
never be updated again, so it was deleted. The 45 that remain are all used,
and some of them (the neutral ramp, the type scale) are candidates to move up
into the theme rather than live here.

## Known fidelity gaps

Both are fonts, and both are visible:

1. The design used more weights of PP Neue Machina (light, medium, ultrabold)
   than the two OTFs the theme vendors. The browser synthesises the rest, so
   thin and heavy text is approximated. Fixing it means adding the font files
   to the theme, not editing this page.
2. `'Biform Pixel'` — the pixel detail in the eyebrow and the marquee — is not
   in the theme and we do not have the file. Those bits fall back to a
   monospace stack.

## How this page is verified

`npm test` proves it compiles, and that is not the same as proving it works.
What compiling cannot catch: the wrong typeface, a broken image, a dead form,
a mobile overflow. So the page is also checked in a real browser against the
deployed URL — computed fonts via `document.fonts`, `naturalWidth` on every
image, `scrollWidth` at 390 px, console errors — plus one real `POST` to the
form endpoint (marked as a smoke test and deleted afterwards) and one deliberate
JS error to prove the error tracker ingests it.

`/version.json` carries the commit the bundle was built from, so "is what
production serves the same as `main`?" is a question with an answer.
