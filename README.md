# Brotea — portfolio

Brotea's public site: what we do, the projects we have shipped, our products,
the team and the contact form. Built with Astro and served as a static site.

It comes from a design artifact, ported to the factory's contracts rather than
pasted in: **the copy is data** (`src/locales/{es,en}.json`, reaching the page
through `t(locale, key)`) and **the colours and fonts are theme tokens**
(`src/styles/theme.css`, the `brotea@2` theme). There is not a single brand hex
or hardcoded string in the page, which is what lets it render in every shipped
language and follow the brand without being edited again.

```bash
npm install
npm run dev      # local dev server
npm test         # what CI runs: build stamp + locale tests + types + build
npm run build    # production build
```

## Languages

Spanish (default, at `/`) and English (at `/en/`). Both are **required**: the
locale test fails the build if a key is missing or empty in either one, so a
half-translated page cannot reach production.

```bash
brotea i18n add <code>     # add a language
brotea i18n check --app .  # what is missing
```

## Structure

- `src/pages/[...lang]/index.astro` — the whole page. One file generates every
  language; the section styles live in its `<style is:global>` block.
- `src/locales/` — the copy, one JSON per language.
- `public/img/` — the images, exported from the design.

## Where it came from

It was a design artifact running on its own runtime, and porting it meant
converting every piece of that runtime into something a browser actually
executes — plus two honest fidelity gaps, both fonts. All of it, and the traps
to avoid next time, is written down in
[docs/porting-the-design-artifact.md](docs/porting-the-design-artifact.md).

## Deploy

Static image behind nginx, on Coolify with `build_pack=dockerfile`. Deploys go
through `scripts/stamp-and-deploy.mjs`, which stamps the commit into
`/version.json` so what production serves can be checked against `main`.
