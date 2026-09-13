---
name: preview-site
description: Start a local PHP server to preview the FrenchNow site and click-test its pages and forms before pushing changes. Use when the user asks to preview, run, or test the site locally.
---

# Preview the FrenchNow site locally

This is a plain HTML/CSS/JS site with PHP form handlers (`newsletter.php`,
`processform.php`) — opening the HTML files directly with `file://` will not exercise
the forms, since the browser can't execute PHP. Use PHP's built-in server instead.

## Steps

1. From the project root, start the server:
   ```
   php -S localhost:8000
   ```
2. Open the live pages in a browser:
   - German (default): `http://localhost:8000/index.html`
   - English: `http://localhost:8000/index_en.html`
   - French: `http://localhost:8000/index_french.html`
   - Russian: `http://localhost:8000/index_rus.html`
3. Click through each page's interactive elements: language-switcher flags, hamburger
   menu, promo-banner close button, any mailto/quick-contact buttons, and submit the
   contact form (DE/FR/RU) and the newsletter form (all pages).
4. Watch the browser console for errors while doing this — several known issues are
   tracked in `AUDIT.md` (root of the repo); check there before assuming a new console
   error is something you introduced.

## Note on the forms

`mail()` will not actually send an email from a typical local machine unless a local
mail server (e.g. `sendmail`/Mailhog) is configured — this is expected. You can still
verify: the PHP script runs without a fatal error, the success/error message text
shown to the user is correct for the page's language, and required-field validation
works. If you need to confirm the exact message that *would* be sent, temporarily add
a `var_dump($message)` / `error_log($message)` line in the relevant PHP file, check the
PHP built-in server's terminal output, then remove it before committing.

## After changing shared markup

Because each language page is a hand-duplicated copy (no shared templates — see
`CLAUDE.md`), re-check the same interaction on all four live pages, not just the one
you edited.
