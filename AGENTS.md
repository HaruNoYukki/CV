# AGENTS.md

Static, dependency-free personal CV/resume site (Ángel García). Three files total: `index.html`, `JS/script.js`, `CSS/styles.css`. No package manager, no build step, no tests, no framework.

## Running

Open `index.html` directly in a browser, or serve statically (e.g. `python3 -m http.server`). There is no build, lint, or test tooling; verification is manual in the browser. Check both light/dark themes and all three languages after any change.

## Architecture

Everything is client-side vanilla JS/CSS:

- `index.html` — single page, default content hardcoded in **Spanish**.
- `JS/script.js` — the `translations` object (es/en/ja) is the source of truth for all section text; `setLanguage()` overwrites the Spanish HTML on load.
- `CSS/styles.css` — theming via CSS custom properties, CRT/terminal visual effects, responsive rules, and a large print stylesheet.

State persists in `localStorage`: `cv_lang` (default `es`) and `cv_theme` (`dark` or absent = light).

## Critical: content lives in two places

Any CV content change (text, jobs, skills, dates) must be made in **both**:
1. The hardcoded Spanish markup in `index.html`
2. The `translations` object in `JS/script.js` — in **all three** language objects (es, en, ja)

The HTML baseline and the es object must be identical (they drifted apart once, e.g. `skill2`/`skill4`/`job1-title`, and were reconciled; treat any new drift as a bug). When translating, es/en/ja objects must stay mutually consistent.

Exception: the most recent job's end date is dynamic. `translations.*.job1Date` stores only the start prefix (e.g. `"Enero 2019 - "`); `updateRecentJobDate()` in JS/script.js appends the current month/year (localized; ja uses `YYYY年M月`), and a `setInterval` re-renders it every minute so an open tab stays current. Hardcoding a full end date anywhere would go stale.

## Adding a translatable element (convention)

Three coupled steps, following the existing id/key mapping:
1. Add an element in `index.html` with an id (kebab-case: `h-about`, `job1-title`, `t-hack`).
2. Add a camelCase key with the same meaning to all 3 language objects (`hAbout`, `job1Title`, `tHack`).
3. Add a `document.getElementById(...).innerHTML = t.key` line in `setLanguage()` (JS/script.js).

## Translation values are raw HTML

Multi-item fields (`job1Desc`, `job2Desc`, `tHack`, `edu1`, `edu2`) are injected with `innerHTML` and **must include their own `<li>`/`<strong>`/`<em>` tags** as single concatenated strings. Plain text fields (`title`, `tAbout`, skill1-6) are plain strings.

## Gotchas

- **Print/PDF is a core feature, ATS-format, exactly one page**: the printer-icon button calls `window.print()`. The print block at the end of styles.css enforces an ATS-friendly PDF: Helvetica everywhere (a `* !important` rule overrides the screen `*` font-family), black-on-white, strict size scale (name 18pt / section titles 14pt / body 10pt), `@page` letter with 6mm/10mm margins, skills in 2 columns, and compact spacing tuned so all three languages' content fits on one page. Verify changes by rendering with headless Chrome (`--headless=new --print-to-pdf=...`) and counting pages. Gotcha: the printed page box is ~740px wide, so the 768px mobile media query activates during print; print rules must explicitly restore horizontal layouts (contact-info, social-links) that the mobile query stacks. When adding UI, add it to the print-hide list; when adding content, re-check the one-page fit and keep the 18/14/10pt scale (see `page-break` rules there).
- **CRT overlay**: `body::after` renders fixed scanlines at `z-index: 9999` with `pointer-events: none`; the body also runs a `crt-flicker` animation. New fixed-position elements will appear *under* the overlay unless given a higher z-index, and should be disabled in print like the rest.
- **Dark mode** works by setting `data-theme="dark"` on `<body>`; colors come only from the `:root` / `[data-theme="dark"]` variable blocks. Use `var(--...)` for any new colors and add a dark variant.
- **Directory names are capitalized** (`JS/`, `CSS/`) and referenced via relative paths (`./CSS/styles.css`). Keep case exact; it matters on case-sensitive hosting (e.g. GitHub Pages). A past commit ("correccion de directorio") fixed broken paths once already.
- **Known placeholder**: the GitHub link in `index.html` is still `href="TU_LINK_GITHUB_AQUI"`.
- Inline `onclick` handlers in HTML (e.g. `setLanguage('es')`, `toggleTheme()`) require those functions to stay global in `JS/script.js`; don't wrap them in modules or IIFEs. Script is loaded at the end of `<body>`; init runs on `DOMContentLoaded`.
- Code comments are written in **Spanish**; match that style.
