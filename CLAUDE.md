# FrenchNow website

Marketing site for Alexandra Descamps, a French teacher offering private lessons in
Switzerland. Plain HTML/CSS/JavaScript with three PHP mail-form handlers. No build
step, no package manager, no framework — every page is a hand-written, self-contained
HTML file.

## Live pages vs. dead files

Only these four pages are actually linked from navigation and listed in
`sitemap.xml` — treat these as the real site:

- `index.html` — German (default/root)
- `index_en.html` — English
- `index_french.html` — French
- `index_rus.html` — Russian

`index_2.html`, `index_engl.html`, and `workbench.html` are **orphaned old drafts** —
not linked from anywhere, not in the sitemap. Don't spend time keeping them in sync;
they're documented as dead in `AUDIT.md` and are candidates for deletion once confirmed
nothing external still points to them. `maintenance.html` is a standalone "coming
soon" page meant to be swapped in manually during downtime — not part of normal
navigation, that's expected.

## Important: no shared templates — content is duplicated per language

Each live page is an **independent copy**, not a template render. There is no shared
header/footer/nav include. This means:

- A fix or content change almost always needs to be applied **by hand, separately, in
  all four live-page files** (`index.html`, `index_en.html`, `index_french.html`,
  `index_rus.html`) — there's no single source of truth.
- The four pages have already drifted from each other in real ways (e.g. the French
  and Russian footers are currently disabled while German's isn't; English is missing
  a contact form the other three have). Don't assume structural parity — check each
  page individually. See `AUDIT.md` for the current known drift.
- IDs, class names, and inline `<script>` logic are copy-pasted per page and have
  already diverged (e.g. one page's mail-button script looks for an id that doesn't
  match what that same page's markup actually uses). When touching any interactive
  element, verify the id referenced in the `<script>` actually exists in *that specific
  file*, not just "on the other pages."

## Where shared code actually lives

- `scripts/main.js` — the only genuinely shared JS, included by all live pages (plus
  the orphans). Handles: first-visit browser-language redirect, the hamburger menu
  toggle, and the promo-banner close button. Everything else interactive (mailto
  buttons, newsletter form submission, language-flag `onclick` handlers) is written
  inline, separately, inside each HTML file.
- `css/main.css` — shared custom styles on top of vendored Bootstrap
  (`css/bootstrap.css` / `css/bootstrap.min.css`, `scripts/bootstrap.bundle.min.js`,
  `scripts/jquery.slim.min.js`, `scripts/parallax.min.js`).
- `newsletter.php`, `processform.php` — the two form handlers actually used by the
  live pages (newsletter signup and the DE/FR/RU contact form, respectively).
  `process.php` is unused dead code — see `AUDIT.md`.

## Known issues

`AUDIT.md` (repo root) is the current, prioritized issue list — broken buttons,
console errors, and cleanup items, with line numbers and a fix-effort estimate. Check
it before assuming a piece of interactive code works; several onclick handlers
reference functions that don't exist. Keep it up to date: when an issue from that list
gets fixed, update its entry rather than leaving it stale.

## Working locally

There's no dev server config — the forms are plain PHP (`mail()`), so a plain
file:// open won't exercise them. Use PHP's built-in server from the project root:

```
php -S localhost:8000
```

Then browse `http://localhost:8000/index.html` etc. `mail()` won't actually send
without local mail configured — this is fine for checking form validation and
redirect behavior, just don't expect a real email to land anywhere in local dev.

## Deploy

Not yet set up — hosting/publish method needs to be confirmed before a deploy skill
or CI step can be built. Ask before assuming FTP, cPanel, or git-based deploy.

## Constraints

- No secrets or credentials belong in this repo — the form handlers only ever need a
  destination email address, nothing sensitive.
- Don't introduce a build step, bundler, or framework migration unless explicitly
  asked — this is intentionally a plain static site with a PHP mail fallback.
