# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A lead-generation **landing-page funnel** for Talkmore (a Norwegian mobile carrier). The user answers a short quiz, leaves contact details, and is entered into a prize draw (`konkurranse`); leads are posted to Airtable and conversions are tracked with the Meta Pixel. All copy is Norwegian (`<html lang="no">`).

There is **no build system, no framework, and no dependencies**. Each page is a single, self-contained `.html` file with all CSS in an inline `<style>` and all JS in an inline `<script>`. The only external assets are images/fonts under `assets/`.

## Three campaign variants

Each variant is a separate, near-identical copy of the same funnel — they share structure, styling, and JS, but differ in prize/copy and backend config:

| File | Campaign | Prize | Quiz |
|------|----------|-------|------|
| `index.html` | `dagligvare` (groceries) | 10 000,– grocery gift card | 4 questions (store → pays? → age → operator) |
| `reise/index.html` | `reise` (travel) | 10 000,– travel gift card | 4 questions (travel agency → pays? → age → operator) |
| `mobilabonnement/index.html` | `mobilabonnement` | mobile subscription for a year | 3 questions (pays? → age → operator) |

**There is no shared/partial include mechanism.** The three files are independent parallel copies that share structure today, but they are **allowed to diverge intentionally** per campaign (e.g. `reise` now has age-based lead-splitting the others don't). Syncing is no longer mandatory: when a change is *meant* to be shared, replicate it by hand across the relevant files and diff afterward; when a change is variant-specific, keep it scoped to that one file. Don't assume every edit must be mirrored across all three.

## Local development & deployment

- **Preview:** serve the repo root over HTTP, e.g. `python3 -m http.server 8000`, then open `http://localhost:8000/` (root), `/reise/`, `/mobilabonnement/`.
- **Do not** open the HTML files directly via `file://`. Every asset and favicon is referenced with a **root-absolute path** (`/assets/...`), including from the subfolder pages, so they only resolve when served from the domain root.
- **Deploy:** copy the files to any static host **at the domain root** (root-absolute `/assets/` paths assume this). There is no CI/CD, linter, or test suite in the repo.
- **Testing is manual:** walk each variant through the funnel in a browser (landing → quiz → form → confirmation, plus the under-18 path).

## Funnel architecture (shared by all variants)

A single-page, screen-switched flow — no routing library.

- **Screens:** `<div class="screen" id="screen-N">`; exactly one carries `.active`. `showScreen(n)` toggles `.active`, scrolls to top, drives the progress bar from `progressConfig`, and (on the confirmation screen) personalizes copy. `goTo(n)` wraps it with `history.pushState` + a `#steg-N` hash; `popstate` and an initial hash check make the browser back button work.
- **Screen ID contract (fixed 1–8):** 1 = landing, 2 = store/travel-agency choice, 3 = "do you pay for the subscription?", 4 = date-of-birth / age gate, 5 = current operator, 6 = contact form, 7 = confirmation, 8 = under-18 disqualification. **The IDs are a contract the shared JS depends on** — `mobilabonnement` intentionally omits `screen-2` and its landing CTA jumps straight to `goTo(3)`, keeping all later IDs identical.
- **Answers:** quiz choices call `answer(key, val)` into a module-level `answers{}` object; it is read at form submission.
- **Age gate (`submitAge`)** builds a Date from the day/month/year selects, validates it, computes age, stores `alder` + formatted `fodseldato`, then routes age < 18 → `goTo(8)`, otherwise → `goTo(5)`.
- **Submission (`submitForm`)** validates name/email/phone + the combined 18-and-consent checkbox, then `POST`s a `fields` object to the Airtable REST API. On success it fires the Meta Pixel `Lead` event and advances to confirmation. **Note:** on Airtable failure it still counts the lead, fires `Lead`, and advances — the user is never blocked by a backend error. Airtable field names are Norwegian (`Navn`, `Epost`, `Telefon`, `Butikk`, `Betaler abonnement`, `Alder`, `Operatør`, `TM Samtykke`, `Dato`) and must match the table schema exactly.
- **Modals** (`privacy-modal`, `rules-modal`) are hash-driven (`#personvern`, `#regler`) via `openModal`/`closeModal` + a `popstate` listener; Esc and overlay-click close them.

## Consent & tracking (GDPR-gated Meta Pixel)

The Meta Pixel does **not** load on page load. The base code (top of `<head>`) defines `loadMetaPixel()` and a safe `firePixelEvent()` wrapper:

- `acceptCookies()` sets the `cookie_consent=accepted` cookie and calls `loadMetaPixel()` (injects `fbevents.js`, inits, fires `PageView`). Returning visitors with the cookie load it immediately.
- `firePixelEvent(name, params)` is a **no-op until consent** and de-duplicates per page session, so calling it without consent (or twice) is safe. The only conversion event is `Lead`, fired from `submitForm`.
- The cookie banner shows only when no `cookie_consent` cookie exists.

## Brand system

Defined once per file as CSS custom properties on `:root` (prefixed `--tm-*`): the green palette (`--tm-green` `#004400`, etc.), accents, neutrals, and the two font stacks. Brand typefaces **CircularXX** (body/titles) and **CircularPro** (CTA buttons) are `@font-face`-loaded from `/assets/fonts/`, with Montserrat (Google Fonts) as the fallback. The color hex values are brand-mandated — don't change them when restyling.

## Per-variant configuration

When editing, know which values are **shared** vs **per-variant**:

- **Shared across all three:** Meta Pixel ID (`1356370719684682`), the Airtable Personal Access Token, and the Airtable `TABLE_NAME`.
- **Per variant (must differ):** the Airtable `BASE_ID` (`appn3POxvI8JlDIb4` root / `appzz6aznFBVs0Rhy` reise / `appEqUjCOOel4Ug20` mobilabonnement), the Pixel `content_name` on the `Lead` event (`dagligvare` / `reise` / `mobilabonnement`), the `<title>`, the landing CTA target (`goTo(2)` vs `goTo(3)`), the quiz screens, and `progressConfig`.

Note: the Airtable PAT and Meta Pixel ID are **live credentials embedded in client-side source** in every file (unavoidable for a static, server-less page). If one is rotated, update all three files together.

## Project memory

Durable facts, decisions, gotchas, and open TODOs accumulate in a living memory
file, imported here so Claude Code loads it automatically. Append to it as the
project evolves.

@.claude/memory.md
