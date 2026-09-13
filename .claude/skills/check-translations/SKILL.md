---
name: check-translations
description: Check the four live FrenchNow language pages (DE/EN/FR/RU) for structural drift after one has been edited — missing sections, mismatched form fields, or ids that no longer match their scripts. Use when the user asks to check translations, sync languages, or verify the language pages are consistent.
---

# Check the language pages for structural drift

The site has **no shared templates**: `index.html` (DE), `index_en.html` (EN),
`index_french.html` (FR), and `index_rus.html` (RU) are independent, hand-duplicated
HTML files (see `CLAUDE.md`). They have already drifted from each other in real ways —
this skill is a maintenance aid to catch further drift, not an auto-translator: it
reports mismatches for a human to review and decide whether they're intentional.

This is a **read-only investigation** — report findings, don't silently "fix" content
across languages without the user's go-ahead, since a structural difference might be
intentional (e.g. different local phone number, different legal footer text).

## What to check, across all four files (`index.html`, `index_en.html`,
`index_french.html`, `index_rus.html`)

1. **Form presence and fields**: does each page have the same set of `<form>` tags
   (contact form action, newsletter form action), and do they submit the same field
   names (`name`, `_replyto`, `message`, `language` for the contact form; `email`,
   `consent` for the newsletter form)? Flag any page missing a form the others have,
   or a field name that doesn't match what the target PHP handler
   (`processform.php` / `newsletter.php`) expects.
2. **Section parity**: do all four have the same major sections (hero/header, about,
   services/pricing, testimonials, contact, footer)? Use heading tags and major
   `id`/`class` landmarks to compare — flag a section present in some pages but
   missing or commented-out in others (e.g. a `<footer>` wrapped in an HTML comment
   in one language but not another).
3. **id ↔ script consistency, per file**: for every `document.getElementById('X')` or
   `querySelector('#X')` in that file's own inline `<script>` blocks, confirm an
   element with `id="X"` actually exists, uncommented, in that same file. Don't assume
   an id that works on one language page also exists on another — check each file
   independently (this is exactly how the FR/RU mail-button bug in `AUDIT.md`
   happened: the id used in DE/EN didn't match what FR/RU's own markup used).
4. **Duplicate ids within a single file**: flag any `id="..."` value that appears more
   than once in the same file — invalid HTML and a latent `getElementById` bug.
5. **Leftover placeholder/template content**: search each file for generic template
   leftovers ("Your Company Name", "Your Logo", "youremail@example.com",
   "Lorem ipsum", template-vendor credit links) that should have been replaced with
   Alexandra's real content, whether currently commented out or live.

## Output

Summarize findings per file (or "no drift found for X"), cross-referencing `AUDIT.md`
so the user can tell what's already known vs. newly discovered.
