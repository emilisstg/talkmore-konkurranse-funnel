# Project Memory — talkmore-konkurranse-funnel

A living scratchpad of durable facts, decisions, and gotchas. CLAUDE.md is the
curated, stable guidance; this file is the evolving notebook — append as the
project changes. Keep entries terse and dated where it helps.

## Durable facts

- **Three parallel variants, no shared includes.** `index.html` (dagligvare /
  groceries), `reise/index.html` (travel), `mobilabonnement/index.html` (mobile
  sub). Any change to shared funnel logic, styles, modals, consent, or brand
  tokens must be hand-replicated across all three.
- **No build system / framework / deps.** Each page is one self-contained
  ~2000-line HTML file: inline `<style>` + inline `<script>`. Only external
  assets are images/fonts under `assets/`.
- **Root-absolute asset paths** (`/assets/...`), used even from the subfolder
  pages → site must be served/deployed at the **domain root**. Local preview:
  `python3 -m http.server 8000` from repo root (not `file://`).
- **Screen-ID contract is fixed 1–8:** 1 landing · 2 store/agency · 3 pays? ·
  4 age gate (DOB) · 5 operator · 6 form · 7 confirmation · 8 under-18 DQ.
  Shared JS depends on these IDs.
- **`mobilabonnement` omits `screen-2`** (no store question) and its landing CTA
  jumps straight to `goTo(3)`; all later IDs stay identical so shared JS works.
  `progressConfig` differs accordingly (1 of 3 @ 33% start).
- **Airtable field names are Norwegian** and must match the table schema exactly:
  `Navn`, `Epost`, `Telefon`, `Betaler abonnement`, `Alder`, `Operatør`,
  `TM Samtykke`, `Dato` (+ `Butikk` on dagligvare/reise).
- **Credentials live client-side in every file.** Airtable PAT + Meta Pixel ID
  (`1356370719684682`) + `TABLE_NAME` are **shared** across all three. Rotating
  either means editing all three files together.
- **Per-variant config:** Airtable `BASE_ID` (`appn3POxvI8JlDIb4` root /
  `appzz6aznFBVs0Rhy` reise / `appEqUjCOOel4Ug20` mobilabonnement), Pixel
  `content_name` (`dagligvare`/`reise`/`mobilabonnement`), `<title>`, landing CTA
  target, quiz screens, `progressConfig`.

## Decisions / gotchas

- **Lead is counted even if Airtable POST fails.** `submitForm`'s `.catch` still
  fires the Meta Pixel `Lead` event and advances to confirmation — the user is
  never blocked by a backend error. Don't "fix" this into a hard failure.
- **Meta Pixel is GDPR-gated.** It does not load on page load; `firePixelEvent`
  is a no-op until `acceptCookies()` runs `loadMetaPixel()`, and it dedupes per
  page session. Calling it without consent or twice is safe by design.
- **Single combined consent checkbox** (`consent-age`) covers both "over 18" and
  the contact/marketing consent; `submitForm` maps it to `TM Samtykke = Ja/Nei`.
- **Brand color hex values are mandated** (`--tm-*` tokens) — don't change them
  when restyling.
- **No tests / linter / CI.** Verification is manual: walk each variant through
  landing → quiz → form → confirmation, plus the under-18 path.

## Conventions

- Edit all three variants together for shared concerns; diff them afterward to
  confirm they stayed in sync.
- Keep Norwegian copy (`lang="no"`) — don't introduce English user-facing text.

## Open TODOs

- (none yet)
