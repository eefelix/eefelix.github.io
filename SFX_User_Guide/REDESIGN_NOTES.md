# SFX-Calc User Guide — Redesign Notes

This document records what was changed in this folder, in what order, and why —
so the reasoning doesn't get lost the next time someone (human or AI) needs to
edit the guide.

## Starting point

`SFX_User_Guide.htm` was a raw Microsoft Word "Save as Web Page" export of
`SFX_User_Guide.docx` (the real source of the wording). That export was:

- ~7,486 lines of Office-generated markup (MSO conditional comments, Word
  CSS classes, inline styles per run of text).
- Backed by `SFX_User_Guide_files/`, a folder of 269 numbered images plus
  Word-internal support files (`colorschememapping.xml`, `filelist.xml`,
  `themedata.thmx`, etc.).
- A single long scroll with no in-page navigation, styled like a printed
  Word document rather than a web page — hard to scan on a phone screen
  inside the app's webview.

The app opens this page (`https://eefelix.github.io/SFX_User_Guide/SFX_User_Guide.htm`,
via the redirect in the repo's `index.html`) in an in-app webview on iOS,
Android and Mac when the user taps the "User Guide" button.

## Request

Redesign the page to look and feel modern and to be easier to navigate
between sections, **without changing the wording of the guide**, while
making sure it still works as a plain static page hosted on GitHub Pages
and renders well inside a phone-width in-app webview.

## Step 1 — Get the content out of the Word export cleanly

Rather than trying to reformat the messy `.htm` export in place, the actual
source of truth — `SFX_User_Guide.docx` — was converted directly:

- Installed `pandoc` (via Homebrew) and converted the `.docx` to semantic
  HTML (`pandoc SFX_User_Guide.docx -f docx -t html --extract-media=...`).
  This produced real `<h1>`/`<h3>` headings, `<table>`/`<tr>`/`<td>`
  structure, and extracted all embedded images as plain numbered PNGs —
  838 lines instead of 7,486, and no Office cruft.
- Installed `beautifulsoup4` in a throwaway virtualenv and parsed that
  output into structured data: one block per top-level section
  (Introduction, Support, Usage → Display, Key, Other features, Menu,
  Limitation).
- The "Key" section (the key-by-key glossary, ~35 entries) is one very
  long HTML table where a "title" row (`(1)` / `All Cancel`, `(2)` /
  `Clear`, …) is followed by one or more detail rows (icon + description).
  A small script grouped those rows back into 35 structured entries
  (number, name, icon(s), description) by detecting the `(N)` pattern —
  this let the entries become individual cards instead of one giant table.
- Two spots where pandoc couldn't convert an embedded Word equation left
  raw, unrendered TeX in the text (`$\sqrt[Y]{X}$` for the Y-th-root
  function, `$1\frac{2}{3}$` for the mixed-fraction example). Both were
  rewritten as plain HTML (`<sup>Y</sup>√X`, `1 ²⁄₃`) — same value, same
  meaning, just actually visible instead of showing literal dollar signs
  and backslashes.

No other wording was changed. Everything else came out of the `.docx`
verbatim.

## Step 2 — Rebuild the page around that content

A single self-contained `SFX_User_Guide.htm` was generated (inline CSS/JS,
no external fonts/CDNs/frameworks), so it can't fail to load or look wrong
because a webview blocked a third-party request:

- **Navigation**: a sticky sidebar table of contents on wide screens that
  collapses into a slide-in drawer behind a hamburger button on narrow
  ones (phones), with the current section highlighted as you scroll
  (scrollspy via `IntersectionObserver`).
- **Key reference**: the 35-entry glossary became a searchable/filterable
  list of cards (type to filter by name or description) plus a row of
  number chips to jump straight to a given key.
- **Other tables restyled as purpose-built components** instead of raw
  Word tables: the display-format/status-bar tables became image+label
  spec cards, the settings table became an iOS-Settings-style list, the
  numbered feature/limitation lists became a plain ordered list and a
  highlighted "limitations" callout.
- **Responsive & theme-aware**: verified with headless Chrome (via the
  DevTools protocol, emulating real device metrics — a plain
  `--window-size` flag turned out to floor at 500px in this Chrome
  version, so accurate mobile screenshots needed
  `Emulation.setDeviceMetricsOverride`) at phone width (390px) and desktop
  width, in both light and dark (`prefers-color-scheme`), confirming no
  horizontal overflow anywhere.
- Images stayed as separate files (not inlined as base64) so the page
  stays small and the browser can cache them; they were copied from
  pandoc's extraction into a new `SFX_User_Guide/media/` folder.
- The old `SFX_User_Guide_files/` folder was deleted — it was Word's
  auto-generated support folder, only ever referenced by the old `.htm`,
  and grep-confirmed to have no other referrers before removal.

## Step 3 — Follow-up refinements from review feedback

1. **Real app icon.** The header/sidebar mark was a placeholder "SFX" text
   badge. Fetched the actual SFX-Calc icon from the App Store listing
   (`https://apps.apple.com/us/app/sfx-calc/id1548427958`) via the iTunes
   lookup API (`itunes.apple.com/lookup?id=1548427958` → `artworkUrl512`),
   saved as `media/app-icon.png` (512px) and a downscaled
   `media/app-icon-128.png` for the header/sidebar badge, and also wired
   up as the page favicon / `apple-touch-icon`.
2. **Bigger images and text.** Initial pass sized things conservatively;
   feedback was that images (calculator-display screenshots, keycap
   icons, the menu screenshot) and body text were too small to read
   comfortably. Increased base font size (root 18px + 1.05rem body, up
   from the browser default 16px) with proportional bumps to headings and
   component text; display screenshots were restructured to stack full
   width under their label (up to 420px) instead of being squeezed to a
   44px-tall strip next to the label; keycap icons grew 34px → 48px, the
   menu screenshot 340px → 520px.
3. **Inconsistent row styling in "Hyperbolic calculation" (and similar
   lookup lists).** Pandoc had promoted just the *first* row of several
   small lookup tables inside key entries (scientific constants, stats
   sums, hyperbolic functions) to a `<thead>`, even though that row isn't
   a real column header — just the first item. That made e.g. "Calculate
   the sinh…" render bold/muted while "Calculate the cosh…" and "…tanh…"
   right below it looked like normal rows. Removed the special header
   styling from these mini-tables so every row in a given list renders
   identically.
4. **Inline combo icons still too small.** The `[ALT] [1]`-style icon
   pairs inside those same lookup tables (e.g. next to "Speed of light …"
   in the scientific-constants list) were still hard to read at 30px
   tall. Increased to 42px.

## Where things stand

- `SFX_User_Guide.htm` is the deployed page (kept at the same filename/URL
  so the app's existing webview link keeps working).
- `media/` holds the key icons, calculator screenshots, and the app icon.
- `SFX_User_Guide_files/` was removed (superseded, see Step 2).
- `SFX_User_Guide.docx` is untouched and still holds the canonical wording.
- Nothing here has been committed to git yet — changes are sitting in the
  working tree for review.

## Important workflow note for future edits

The page is now hand-built HTML, not a raw Word export. **Re-exporting
`SFX_User_Guide.docx` as a web page from Word will *not* regenerate this
design** — it would just produce another messy Office export.

For future content changes, either:

- Edit `SFX_User_Guide.htm` directly (it's a single plain HTML file with
  inline CSS/JS), or
- Update `SFX_User_Guide.docx` with the wording change and ask for the
  page to be regenerated through the same docx → pandoc → structured
  rebuild pipeline described above, so the `.docx` stays the source of
  truth for wording.

## Not yet done / open item

`SFX_User_Guide/Examples/*.mp4` contains several demo videos (percentage
calculation, engineering/SI mode, pi/equal-key swap, iPad app-switch
bug/fix) that exist in the repo but aren't linked from the guide at all.
Flagged during the redesign but intentionally left alone since it wasn't
part of the request — worth a "Video walkthroughs" section if wanted
later.
