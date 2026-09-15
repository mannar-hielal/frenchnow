# FrenchNow website — issue list & cost estimate

*Prepared for Alexandra and for internal dev planning. Nothing on the live site has
been changed yet — this is a review to go through together first.*

Issues are grouped by priority:
- **Prio 1** — things that are actively broken right now (errors, non-working buttons).
- **Prio 2** — not broken, but should be fixed (visible gaps, slow pages, business risk).
- **Prio 3** — cleanup / nice-to-have, no visitor ever notices these.

---

## Prio 1 — currently broken

### 1. The language flags (DE/FR/RU/EN switcher) don't actually do anything when clicked
Every time a visitor clicks one of the little flag icons to switch language, the site
tries to run a piece of code that was never finished. It still navigates to the right
page because the link itself works, but it silently fails in the background on
**every page, every time**. If we ever want the flags to show which language is
currently active (e.g. highlight the current one), that part has simply never worked.

*Technical: `onclick="setActiveLanguage(...)"` is used on every language link across
all pages, but the function `setActiveLanguage` is never defined anywhere in the
codebase (checked `scripts/main.js` and every inline `<script>`). Throws
`ReferenceError: setActiveLanguage is not defined` in the browser console on every click.*

### 2. ~~The envelope/mail button does nothing on the French and Russian pages~~ FIXED
~~On `index_french.html` and `index_rus.html`, the small red envelope button next to
the phone icon (top of the page) is supposed to open an email to Alexandra when
clicked. It currently does nothing at all.~~

**Fixed:** the click handler in both files (`index_french.html:462`,
`index_rus.html:462`) now calls `getElementById('mailbutton2')`, matching the actual
button id in the markup (`index_french.html:154`, `index_rus.html:154`). Button is
wired up correctly and opens `mailto:alex.descamps@outlook.com`.

### 3. The homepage throws an error on every single visit
Not visible to a normal visitor, but every time anyone loads the German homepage
(`index.html`), the browser logs an error because of leftover code from an older
version of the newsletter signup box.

*Technical: `index.html:640` attaches a `submit` listener to
`document.getElementById('newsletterForm')`, but the only `newsletterForm` markup in
this file is commented out (dead code around line 555). `getElementById` returns
`null` → `TypeError` on every load.*

### 4. The homepage contains an entire hidden second web page pasted inside it
Somewhere along the way, a whole leftover template snippet (a "footer" example, titled
"Classic Three-Column Footer") got pasted directly into the middle of `index.html`
instead of just the small bit that was needed. The page currently contains two
complete, overlapping documents stacked on top of each other.

*Technical: `index.html` lines 707–931 contain a second, complete
`<!DOCTYPE html><html>...</head><body>...</body></html>` document nested inside the
real page, followed by a stray extra `<script>AOS.init()</script>` and duplicate
closing `</body></html>` tags (lines 935–940). Net result: 2 `<html>` opening tags vs.
3 closing tags, 2 `<body>` opens vs. 2 closes — invalid HTML that risks broken
rendering in some browsers and confuses search engines and screen readers.*

---

## Prio 2 — should be fixed

### 5. The French and Russian pages have no footer at all
Unlike the German homepage (which has a footer with a copyright line, a "back to top"
button and a LinkedIn link), the French and Russian pages currently show **no footer
section whatsoever** — it was disabled at some point and never re-enabled.

*Technical: the entire `<footer>` block is wrapped in an HTML comment in both
`index_french.html` (lines 424–440) and `index_rus.html` (lines 424–440), byte-for-byte
identical in both files. The commented-out block itself also still contains leftover
placeholder text ("©Your Company Name. All rights reserved.", a "Krakatoom-Design"
template-maker credit) — good that it's currently hidden, but if anyone re-enables the
footer without cleaning that text first, the placeholder text would go live.*

### 6. The English page has no way to send a written message — only a newsletter signup
`index.html`, `index_french.html` and `index_rus.html` all have a full contact form
(name, email, message) that emails Alexandra directly. `index_en.html` only has two
"mailto" buttons (which open the visitor's own email app) and a newsletter signup —
there is no equivalent message form for English-speaking visitors.

*Technical: `index_en.html` has no `<form action="processform.php">` at all —
confirmed by searching for any form tag in the file; the other three live pages all
have one.*

### 7. Several homepage photos are much larger than they need to be
A handful of images are 2–7 MB each, which is far more than a browser needs to display
them at their actual on-page size. This slows down page load for every visitor,
especially on mobile / weaker connections — and can hurt Google ranking.

*Technical: `images/BildBusiness.jpeg` (6.7MB), `images/BildMitte.jpeg` (4.8MB),
`images/learn-french-in-paris_1920x800min.jpeg` (4.3MB), `images/BildHeader.jpeg`
(4.3MB), `images/alex_decamps.png` (3.7MB), `images/BildKontaktFormular.jpeg` (2.6MB),
and a few more in the 1–2MB range. Simple re-export/compression (no visible quality
loss) would typically cut these by 80–95%.*

### 8. Duplicate technical IDs on the contact form pages
Not visible to a visitor, but `index.html`, `index_french.html` and `index_rus.html`
each use the same internal identifier (`id="email"`) twice on the same page — once on
a decorative, disabled field near the top, and once on the real contact-form email
field. This is invalid HTML and a latent bug: any future script that looks up
`#email` would only ever find the first (decorative, disabled) one.

*Technical: `index.html:190` & `index.html:532`; `index_french.html:153` &
`index_french.html:373`; `index_rus.html:153` & `index_rus.html:373`. `index_en.html`
does not have this duplicate.*

### 9. The "contact us sent successfully" page mixes two incompatible versions of a library
When a visitor submits the contact form, they land on `processform.php`, which loads
two different, incompatible major versions of the same UI library (Bootstrap 5 CSS
together with Bootstrap 4 JavaScript). Any interactive Bootstrap component on that
result page (dropdowns, collapse menus, etc.) can behave unpredictably. There's also a
small PHP warning logged on the server if that page is ever opened directly rather
than via a form submission.

*Technical: `processform.php` loads `bootstrap@5.3.2` CSS from a CDN but
`bootstrap/4.5.2` JS from another; Bootstrap 4 uses `data-toggle`, Bootstrap 5 uses
`data-bs-toggle` — incompatible event wiring. Also, `$language` is read via
`<?php echo $language; ?>` outside the `if ($_SERVER['REQUEST_METHOD'] === 'POST')`
block where it's actually set, causing an "undefined variable" notice on a plain GET
request to that URL.*

### 10. Leftover duplicate code in the shared script file
`scripts/main.js` has the same function (`redirectToLanguagePage`, the logic that
sends first-time visitors to their browser's language) written out **twice** — an
old, slightly-wrong version and a fixed version right after it. It works today only
because the second copy happens to override the first, but it's fragile: anyone
editing the first copy by mistake (because it's listed first) would have no effect at
all, silently.

*Technical: `scripts/main.js` lines 10–41 (old copy, references non-existent
`index_engl.html`/`index_en.html` inconsistently) and lines 73–102 (corrected copy,
this one wins because it's declared later). Should be reduced to a single copy.*

---

## Prio 3 — cleanup / good to have (no visitor impact)

### 11. Old page copies still sitting in the project
`index_2.html`, `index_engl.html` and `workbench.html` are older drafts/scratch copies
of the real pages. They are **not linked from anywhere** on the live site and are not
listed in `sitemap.xml`, so no visitor and no search engine will ever reach them —
they're just taking up space and could confuse a future developer. Recommend archiving
or deleting once we confirm nothing external (an old bookmark, an old ad, an old QR
code) still points to them.

### 12. An entire unused form-handler file, referencing an image that no longer exists
`process.php` is not used by any form on any current page — nothing on the live site
points to it. It also references `images/zap-bestehen-qr.png`, a file that no longer
exists in the `images/` folder, so even loaded directly it would show a broken image.
Safe to delete.

### 13. Various old commented-out code blocks
Several pages (`index.html`, `index_french.html`, `index_rus.html`, `index_engl.html`)
have sizeable chunks of old HTML/JavaScript left commented out rather than removed
(old newsletter form markup, an old French/Russian "language toggle" function, an old
contact-form handler in `process.php`). Harmless, but worth tidying next time someone
is already working in these files.

---

## ✅ Resolved: where form submissions go

Alexandra confirmed all form submissions (contact form and newsletter signup) should
go to a single address: `alex.descamps@outlook.com`. `newsletter.php` previously sent
to a different, unconfirmed address (`swissperspective@gmail.com`) — this has been
updated to `alex.descamps@outlook.com`. `process.php` and `processform.php` already
used the correct address and needed no change. `swissperspective@gmail.com` is no
longer referenced anywhere in the active codebase.

---
