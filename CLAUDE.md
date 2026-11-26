# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Static Czech-language website "NIC – Not In Coins" (facts about the Czech koruna, banknotes, a timeline, a clicker game) served at https://niclol.eu via GitHub Pages (`CNAME`). There is no build step, package manager, linter, or test suite: every page is a self-contained, hand-edited `.html` file with inline `<style>` and `<script>`. To preview, open the file in a browser or serve the folder (e.g. `python -m http.server`) — use a server, since pages use root-absolute paths (`/img/...`, `/favicon.png`).

All user-facing text is Czech (`lang="cs"`); keep it that way. Commit messages are written in Czech without diacritics.

## Architecture: duplicated page template

Pages (`index`, `bankovky`, `casova-osa`, `hra`, `ledomrunkalis`, `kontakt`) were split out of a former single-page app and still share one template copied into each file:

- **Same `<head>` boilerplate**: clean-URL script, legacy hash redirect, SEO/OG/Twitter meta, Google Fonts (Cormorant Garamond + Jost).
- **Nearly identical CSS** (color tokens on `:root`, `.page`/`.page.show` sections, header/nav, cookie banner). `index.html` and `ledomrunkalis.html` have small page-specific additions.
- **Identical header nav** (`<nav>` with `data-p` keys) and cookie banner markup.
- **One identical JS bundle at the end of every page**, with sections `/* data */`, `/* grafika */`, `/* kurzor */`, `/* hra */`, `/* ledomrunkalis */`, `/* cookies */`, plus Google Analytics (`GA_ID`), which loads only after analytics consent stored in `localStorage` key `nic_cookie_consent`.

Each file contains only its own `<section class="page show" id="...">` content, but runs the full bundle. That is safe because the helper `$` returns a no-op `Proxy` (`D`) when `document.querySelector` finds nothing, so code for elements on other pages silently does nothing. Keep using `$` (not raw `querySelector`) in shared code for this reason.

**Consequence:** a change to the nav, header, cookie banner, shared CSS, or JS bundle must be applied to **all six page files** to keep them in sync. Check with grep/diff after editing.

Banknote data lives in the `N` array in the JS bundle (`OLD` = discontinued notes, `EV` = timeline events, `TR` = banknote outline paths); images are `img/<value>.webp` (e.g. `img/100.webp`, 1000 px wide), built from `n.v` in templates. Timeline backgrounds are `img/rytina-*.webp`. The original `.jpg` files are kept only so old external links keep working.

**Pre-rendered content (SEO):** `#nl` and `#old` in `bankovky.html` and `#tl` in `casova-osa.html` contain static HTML generated from the bundle templates, so crawlers see the text without running JS. The templates render only when the container is empty (`if(!$("#nl").firstElementChild)...`, `renderTl` returns early). **When you change `N`, `OLD`, `EV` or the templates, update the static HTML in those pages too** — otherwise the page keeps showing the old text. `N[i].fun` (click toasts) is still read from JS. In `bankovky.html`, `TR` is a stub because the outlines are already inline in the SVGs.

## URLs and SEO

- Clean URLs without `.html` (`/bankovky`, `/casova-osa`, ...). A head script rewrites `/x.html` and `/index.html` to the clean form, and old SPA hash links (`/#hra`, `/#osa`, ...) redirect via a key → path map in `index.html`. If you add or rename a page, update that map, the nav in every page, and `sitemap.xml`.
- Each page has `canonical`, `og:*` and `twitter:*` meta with **absolute** `https://niclol.eu/...` URLs; the OG image is `og-default.png` (or a page-specific one such as `og-ledomrunkalis.png`, 1200×630).
- JSON-LD (`application/ld+json`) appears in `index.html`, `ledomrunkalis.html` (including FAQ schema), `bankovky.html` and `casova-osa.html` (`BreadcrumbList` + `ItemList`). The last two repeat the text of `N` and `EV`, so update them together with that data.
- Scroll-in animations (`[data-a]`) hide content only under the `js` class set by the first head script, so write those CSS rules as `.js [data-a]...`. Keep `<h1>` and its wrapper free of `data-a` (LCP).
- When a page's content changes, bump its `<lastmod>` in `sitemap.xml`.
- `404.html` is standalone (own minimal styles, `noindex`) and does not use the shared template.
