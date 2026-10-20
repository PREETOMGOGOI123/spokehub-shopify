# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working mode: guide-only, no direct edits

The user is learning Shopify Liquid theme development from scratch (goal: contract/freelance jobs) and wants to write the theme code themselves. **Do not use Write or Edit on theme files unless the user explicitly asks you to make the change directly.** Default behavior instead:

- Explain the concept/fix and give the exact code needed.
- Tell them precisely which file and where it goes (e.g. "in `sections/header.liquid`, inside the `{% schema %}` block").
- Let them paste it in themselves.
- Keep replies short and go one step at a time rather than dumping a full solution.

This preference has flipped before — if the user says something like "just edit the files directly," follow that instead until they say otherwise.

## What this is

A from-scratch Shopify theme ("Cycling Theme") for a bicycle shop (spokehub), built manually rather than off Dawn/Horizon or the Shopify CLI scaffold. It intentionally omits pieces a full Shopify theme normally has:

- No `locales/` directory — no translation files, strings are hardcoded in sections.
- No `blocks/` directory — sections do not use the blocks API.
- No JS in `assets/` — the only script in the theme is a small inline `<script>` in `sections/main-product.liquid` for thumbnail swapping.
- One global stylesheet (`assets/base.css`, ~580 lines) rather than per-section CSS files. All new styles go here.
- `config/settings_schema.json` only has `theme_info` — no global theme settings section exists yet.

Don't "fix" these omissions (e.g. don't add a locales system or split CSS per-section) unless asked — they're a deliberate simplification for a learner-built theme, not oversights.

## Architecture

Standard Shopify Online Store 2.0 JSON-template structure:

- `layout/theme.liquid` — the only layout file. Extremely minimal: loads `base.css`, renders `{% section 'header' %}`, `{{ content_for_layout }}`, `{% section 'footer' %}`. No cart drawer, no JS bundle, no font loading logic.
- `templates/*.json` — each template is just an ordered list of section references (e.g. `templates/index.json` wires `hero` → `category-circles` → `featured-products` → `service-banner` → `service` → `testimonials`). To reorder or add sections on a page, edit the template JSON, not the section files. `templates/collection.json` prepends `category-circles` above `main-collection`; `templates/gift_card.liquid` is the one Liquid (non-JSON) template.
- **Alternate templates per resource**: products can use either `product.json` (→ `main-product` section) or the suffixed `product.bikes.json` (→ `main-product-bikes` section, a bike-specific PDP that surfaces the `custom.*` Marin metafields). A product is assigned the `.bikes` variant via the "Theme template" dropdown in the Shopify admin. When editing product-page markup, check which of the two sections applies — they are separate files, not shared.
- `sections/*.liquid` — one section per page area, each ending in its own `{% schema %}` block defining that section's settings (text/image/link_list inputs surfaced in the theme editor). Sections are self-contained: markup + schema in one file, no companion CSS/JS file.
- `snippets/*.liquid` — small reusable partials rendered via `{% render 'name', ... %}` (e.g. `product-card.liquid` used by both `main-collection.liquid` and `featured-products.liquid`, `service-card.liquid` used by `service.liquid`).
- `assets/` — images referenced via `{{ 'file.jpg' | asset_url }}`, plus the single `base.css`.
- `config/settings_schema.json` / `settings_data.json` — theme-wide settings; currently only `theme_info` is populated.
- `.shopify/metafields.json` — local cache of the store's metafield definitions (mostly Shopify's built-in product category metafields, plus several `custom.*` fields for a "Marin" bike product line: story content, geometry pages, collection references). This file is gitignored-equivalent locally (`.shopify/.gitignore` excludes the whole directory from the CLI's own tracking) but is checked in here for reference — treat it as read-only documentation of what metafields exist in the store, not something to hand-edit.

## Conventions observed in the existing code

- BEM-style class names throughout (`header__nav-link`, `card__badge`, `product__option-values`, `filter__price`), block name matching the section/snippet (`.card` in `product-card.liquid`, `.product` in `main-product.liquid`). Follow this when adding markup.
- CSS custom properties define the whole design system at the top of `base.css` (Radix-derived spacing/radius/type/color scale: `--space-*`, `--radius-*`, `--font-size-*`, `--gray-*`, `--red-*` as the brand accent, `--shadow-*`). Use these tokens instead of hardcoded pixel/hex values in new CSS.
- Native HTML/CSS for interactive UI where possible instead of JS: the mobile nav uses a hidden checkbox + label (`#nav-toggle`), the collection filters toggle the same way (`#filters-toggle`), collection filtering/pagination uses Shopify's native `collection.filters` and `{{ paginate | default_pagination }}` rather than AJAX.
- Icons are inline SVG (stroke-based, `currentColor`), not an icon font or sprite sheet.
- Schema blocks are kept minimal — only add settings that are actually used in the section's markup.

## No build tooling

There is no `package.json`, bundler, or test suite in this repo — it's pushed straight to Shopify. Preview/deploy with the Shopify CLI (`shopify theme dev` / `shopify theme push`) if the user has it installed; don't introduce a build step unless asked.

The one piece of tooling present is `.theme-check.yml` (Shopify's Liquid/theme linter config) — it currently disables the `ImgWidthAndHeight` rule. Run `shopify theme check` if the user wants lint feedback.
