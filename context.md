# Shopify Custom Theme Migration Context

**Project Goal:** Migrating a static, custom-designed, single-product HTML/CSS website into a Shopify Dawn theme. We are bypassing the standard Dawn structures to create a highly bespoke layout, section by section.

## Progress So Far

We have successfully migrated the foundation and the first two core sections of the website:

1. **Global Architecture:**
   - Stripped the default Shopify `class="gradient"` from the `<body>` tag in `layout/theme.liquid`.
   - Linked custom Google Fonts (Bebas Neue & Space Grotesk) in `theme.liquid`.
   - Created `assets/custom-global.css` for root design system variables (`--bg-dark`, `--cyan`, `--magenta`, etc.) and forced them on the `body` tag using `!important` to prevent Dawn overrides. Linked this globally in `theme.liquid`.
   - Renamed all image assets in the `assets/` folder to replace spaces with hyphens to satisfy Shopify's strict naming conventions.

2. **Navbar (`sections/header.liquid`):**
   - Replaced Dawn's default header with the custom HTML structure.
   - Extracted its CSS into `assets/custom-header.css` and linked it natively.

3. **Hero Section (`sections/custom-hero.liquid`):**
   - Extracted the Ambient layer, interactive orbs, and product stage.
   - Migrated the 750+ line CSS block to `assets/custom-hero.css`.
   - Updated `templates/index.json` to set `custom_hero` as the primary active section.
   - **Resolved rendering blockers:** Fixed invisible background grids and orbs by forcing a local stacking context, and bypassed Theme Editor JavaScript blockers by hardcoding CSS animation trigger classes.

## Coding Format & Best Practices for Next Steps

When migrating the remaining sections (Wing/Stats, Music, Productivity, Setups, Design, Socials), strictly follow this workflow:

1. **Section Isolation:**
   - Create a new liquid file for each section (e.g., `sections/custom-wing.liquid`).
   - Copy the relevant HTML from the original `index.html`.
   - Wrap it with a minimal Shopify schema:
     ```liquid
     {% schema %}
     {
       "name": "Custom [Section Name]",
       "presets": [{"name": "Custom [Section Name]"}]
     }
     {% endschema %}
     ```

2. **CSS Extraction:**
   - Create a corresponding CSS file (e.g., `assets/custom-wing.css`).
   - Extract _only_ the CSS for that specific section from the original `style.css`.
   - Ensure the extracted CSS is perfectly valid (no broken syntax during extraction, as it will break all subsequent styling/animations).
   - Link it at the very top of the liquid section using:
     `{{ 'custom-wing.css' | asset_url | stylesheet_tag }}`

3. **Asset References:**
   - All images must use Shopify's liquid filters. Replace standard `<img src="image.png">` with:
     `<img src="{{ 'image-name.png' | asset_url }}">`
   - _Crucial:_ Ensure you use the hyphenated image names (e.g., `Colored-Desk-Set-Up.png`), as spaces will cause Shopify to throw an "illegal characters" error.

## Critical Rules & "Gotchas" Do Not Miss:

- **Dawn Theme Interference (Stacking Contexts):** Dawn's `base.css` and `theme.liquid` classes are aggressive. Shopify sections have transparent backgrounds by default. If your custom section relies on absolute background elements (like `ambient-layer` with `z-index: 0`), they will be invisible. **Fix:** Explicitly assign a `background-color` (e.g., `var(--bg-dark)`) and `z-index: 1` to the top-level section wrapper (`.hero-wrapper`).
- **Theme Editor JS Blocking:** The Shopify Theme Editor runs in an isolated iframe that actively blocks or immediately reverts global `document.body` manipulations via JS. **Fix:** Do not rely on JavaScript to add load classes (e.g., `document.body.classList.add('hero-loaded')`). Instead, hardcode the trigger class (e.g., `<div class="hero-wrapper hero-loaded">`) directly onto the section wrapper so entrance animations trigger purely via CSS.
- **Section Visibility:** After creating a new section, it will not appear on the homepage automatically. It must be manually added to `templates/index.json` or added via the Shopify Theme Customizer dashboard.
