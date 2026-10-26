# Sponsor a Need — PRD

## Original problem statement
Public website "Sponsor a Need" connecting donors directly with small children's homes in India. Google Sheets CSV = database, n8n production webhook = form submission. No DB, no auth, no payments, no admin panel.

## Latest direction (Aug 2026, consolidated 15-point update)
STRICT SINGLE-SPONSOR model: "One person. One need. One complete impact." Each need has exactly one sponsor who covers the full amount. No contributor/partial language anywhere. Storytelling-first experience, Telugu toggle, multiple-home tabs, face-free imagery, dynamic impact tiles.

## Architecture
- React SPA with TRUE multi-page routing (react-router-dom v7 BrowserRouter), all logic in /app/frontend/src/App.js; styles in /app/frontend/src/App.css. Backend unused by design.
- Routes: / (Home: hero, motto band, story teaser, featured needs, impact preview, fulfilled preview, quote, CTA) · /explore-needs (full needs experience: home tabs, search, category/location/status filters, sort) · /needs/:needId (details page: story, why it matters, impact copy, sponsor modal) · /why-we-exist (why + storytelling + trust) · /how-it-works (one-need-one-sponsor block + 6-step journey) · /our-impact (stats, journey, tiles, fulfilled, quote) · /track-my-sponsorships · /homes/:homeName (dynamic children's home pages).
- Shared Layout (nav with active-state NavLinks + language toggle + footer) via Outlet; ScrollToTop + page fade on route change. Site data + language via React context (SiteCtx).
- Data: GET https://docs.google.com/spreadsheets/d/e/2PACX-1vTa0GnC0EIzhnHAoScX5MWXFAEUy_9UI5FNShaOHrcpB9xhIRRY5QvtJvNXbS73sjnnO2KYwzvKssZh/pub?gid=0&single=true&output=csv (published-as-CSV URL since 2026-08-25; fetched with cache:"no-store" + _ts cache-buster)
  - Live schema now: Need_id, Home_Name, Need_Title, Description, Amount, Status, Location, Created_At, Sponsor_Name, Sponsor_Contact, Update_Note. Code reads amount via `need.Amount ?? need.Target_Amount` (both schemas safe).
- Mutation: POST https://ramyachil3.app.n8n.cloud/webhook/eb2f249c-7eb3-48e7-8b55-557427e6642b (original spec URL 28b27c70... is dead/inactive — switched with evidence). Payload EXACTLY: {Need_id, Sponsor_Name, Sponsor_Contact, Message}. Need_id is dynamic from the selected card.
- i18n: COPY dictionary (en/te) in App.js; only UI strings translated, sheet data raw; choice persists via localStorage("san-lang").
- Imagery (verified face-free via AI analysis): Pexels 20616993 (schoolchildren from behind), 6641892 (child with backpack, back view). Rejected for visible faces: 31790049, 11604297, 15813606.

## Implemented (2026-08-24)
- Single-sponsor UI: cards show total amount + status only; modal CTA "Sponsor this" (open), disabled "Already Sponsored" (sponsored), "Fulfilled" note (fulfilled, no form).
- Status filter bug FIXED (was comparing whole object; now trims + lowercases need.Status).
- Home tabs (All Homes + dynamic per-home tabs with open-need counts + home intro line).
- "Behind every need is a story" section (large type, before/after cards, image collage) + quote band ("Today's children, tomorrow's future. Let's build the future together.").
- Impact stats reworked for single-sponsor: homes, needs waiting, sponsored in full, fulfilled, ₹ value fulfilled + 6-step journey.
- Recently Fulfilled: dynamic impact tiles (count, ₹ value, homes) + cards — real sheet data only.
- English | తెలుగు toggle (nav, all sections, forms, messages; persists across reload; desktop + mobile).
- Track My Sponsorship preserved (contact match, timeline, fulfilled note, total amount).

## Verification (Playwright, 2026-08-24) — all 20 spec checks
- STALE-CACHE FIX (2026-08-25): sheet fetch now uses `cache: "no-store"` + `_ts=Date.now()` cache-buster. 11 checks PASS — request confirmed cache-busted; real sheet shows N001 Open; simulated n8n flip (MOCKED CSV, sheet not touched): N001 leaves Open list, appears under Fulfilled, detail page shows fulfilled note with NO sponsor button, other Open needs unaffected, impact tiles update (1 fulfilled / 20,000). NOTE: fix is in PREVIEW code — production (homes-needs.emergent.host) needs a redeploy. Caveat: Google's published CSV itself can lag sheet edits by a few minutes (Google-side publishing delay, not the website).
- MULTI-PAGE (2026-08-24): 23 checks PASS — direct loads on all routes, active nav highlight, /needs/N001 details + sponsor form, /homes/<name> dynamic pages, 6-step journey, Telugu persists across all pages, tracker intact on its own page, home page previews render.
1-5 filters (All/Open/Sponsored/Fulfilled) PASS. 6 correct need opens PASS. 7 Need_id=N002 captured in live POST PASS. 8/9 name+contact PASS. Payload = exactly 4 spec fields PASS. 10/12 real submission succeeded with acknowledgment PASS. 13 tracker PASS. 14 home tabs PASS. 15 Telugu toggle + persistence PASS. 16 real-data empty states PASS. 17 storytelling/quote render + images load PASS. 18 mobile menu + toggle PASS. 19 no identifiable faces (AI-verified images) PASS. 20 existing features intact PASS.
- Check 11 (Sponsored need can't be re-sponsored): code path implemented (disabled "Already Sponsored" button) but NOT exercised live — sheet currently has no valid Sponsored row. Verify after n8n marks a need Sponsored.
- Note: no `testing_agent` subagent exists in this environment; verification was done with the browser automation tool incl. network payload capture.
- Two test submissions sent to the live webhook ("Website E2E Test", "Website E2E Test 2") — user may delete from sheet/Telegram.

## Known follow-ups
- n8n write-back to Google Sheet unverified from outside (rows didn't change after live tests) — user to check n8n Executions.
- When sheet switches to new columns, site adapts automatically (Amount fallback in place).

## Backlog
- P1: Verify "Already Sponsored" gating once a valid Sponsored row exists.
- P2: Per-home about/story content (needs a sheet column or user-provided copy — not fabricated).
- P2: Multiple matches in tracker if one contact sponsors several needs.
