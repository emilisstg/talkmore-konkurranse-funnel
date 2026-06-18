# Project Memory — talkmore-konkurranse-funnel

A living scratchpad of durable facts, decisions, and gotchas. CLAUDE.md is the
curated, stable guidance; this file is the evolving notebook — append as the
project changes. Keep entries terse and dated where it helps.

## Durable facts

- **Three parallel variants, no shared includes.** `index.html` (dagligvare /
  groceries), `reise/index.html` (travel), `mobilabonnement/index.html` (mobile
  sub). They share structure today but are **allowed to diverge intentionally**
  per campaign — syncing is no longer mandatory. Replicate by hand only when a
  change is *meant* to be shared; keep variant-specific changes scoped to the one
  file (e.g. `reise` has age-based lead-splitting the others don't).
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
- **`reise` only: age-based lead split at submit (2026-06).** `submitForm`
  recalculates age from the DOB dropdowns and routes 70-and-over leads to a
  second Airtable base (`AIRTABLE_BASE_OVER70` = `appap8dLTVgS183Pj`, same
  `TABLE_NAME`/PAT as main) and to a dedicated **in-page** ending (`screen-9` /
  `#steg-9`, via `goTo(9)` — no external redirect) with **no** Meta Pixel `Lead`
  event; over-70s must never be pixeled (keeps the pixel off old, low-converting
  leads). Under-70 flow is unchanged. `finishSubmission(isOver70, consentTM)`
  centralizes the success/non-blocking-failure completion (both branches call
  it). 70+ are still entered in the draw (legal compliance), just untracked.
  This adds `screen-9` beyond the 1–8 ID contract — a deliberate reise-only
  divergence. dagligvare & mobilabonnement do NOT have this — don't back-port
  unless asked.

## Conventions

- Edit variants together only for genuinely shared concerns, then diff them to
  confirm intended parity; don't mirror variant-specific changes by reflex.
- Keep Norwegian copy (`lang="no"`) — don't introduce English user-facing text.

## Open TODOs

- (none) — `reise` over-70 split is live: real base `appap8dLTVgS183Pj` wired in,
  in-page `screen-9` ending, shared PAT confirmed to have write access to it.
