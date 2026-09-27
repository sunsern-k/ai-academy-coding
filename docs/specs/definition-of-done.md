# Definition of Done — Truck2Hand

Source: Confluence [Definition of Done](https://tripetchgroup-cdo.atlassian.net/wiki/spaces/TBW/pages/5143265326/Definition+of+Done) (v12). The release is done only when **every** box below is true.

## 1. Stack (exact versions — pin without `^` or `~`)

All versions below were confirmed to exist on the npm registry (`npm view <pkg>@<version>`, checked 2026-09-27). `@vitejs/plugin-vue@5.2.4` declares peer `vite ^5 || ^6`, and `better-sqlite3@12.1.0` declares `engines.node` including `24.x`, so the set is compatible.

**Runtime:** Node.js `24.20.0` (commit `.nvmrc` and `"engines": { "node": "24.20.0" }`).

**Frontend — Vue.js only**

| Package | Version |
| --- | --- |
| vue | 3.5.42 |
| vue-router | 4.6.4 |
| vite | 6.4.3 |
| @vitejs/plugin-vue | 5.2.4 |

**Backend — Node + Express only**

| Package | Version |
| --- | --- |
| express | 5.2.1 |
| drizzle-orm | 0.44.7 |
| better-sqlite3 | 12.1.0 |
| cors | 2.8.6 |
| multer | 2.3.0 |
| swagger-ui-express | 5.0.1 |

Rules for agents:
- Do not add other runtime dependencies. If one seems necessary, stop and ask.
- `drizzle-kit` is not in the list, so create tables with an idempotent SQL bootstrap (`CREATE TABLE IF NOT EXISTS …`) run at startup; Drizzle is used for schema definitions and queries.
- Tests use the built-in `node:test` runner (no extra dependency). See REQUIREMENTS Q7.

## 2. Checklist

- [ ] **DOD-1** Frontend is built with Vue.js using the versions above.
- [ ] **DOD-2** Backend is Node + Express using the versions above.
- [ ] **DOD-3** Frontend is responsive (breakpoints in REQUIREMENTS § Look and feel).
- [ ] **DOD-4** The three pages — all products, product detail, product management — work and link to each other.
- [ ] **DOD-5** Multiple filters can be applied at the same time (AC-1.9).
- [ ] **DOD-6** A listing's cover image can be changed (AC-3.13).
- [ ] **DOD-7** Forms have validation and the system has error handling (AC-V.*), including:
  - [ ] duplicate listing check — titles must be unique (AC-V.2)
  - [ ] unreasonable price — negative or greater than 99,999,999 (AC-V.3)
  - [ ] prohibited / illegal listings (AC-V.4)
- [ ] **DOD-8** Sort by newest, price and most clicked/viewed (AC-1.10, AC-2.10).
- [ ] **DOD-9** The team can demo the flow: **view products → filter → view detail → add / edit / delete product** (script below).
- [ ] **DOD-10** The team can explain which parts AI helped with and what humans reviewed or changed (`docs/AI-USAGE.md`, see below).
- [ ] **DOD-11** Backend has unit tests; `npm test` in `backend/` passes.

## 3. Engineering gates (derived from the checklist)

- [ ] `npm install && npm run dev` works from a clean clone in both packages; `README.md` documents it.
- [ ] Backend seeds lookup data on first start and serves Swagger UI at `http://localhost:8080/api-docs` covering every endpoint.
- [ ] Every AC tagged `unit` or `api` has at least one test referencing its ID in the test name, e.g. `test('AC-V.3 rejects negative price', …)`.
- [ ] Service-layer validation rules have unit tests independent of HTTP.
- [ ] No console errors on the three pages during the demo flow.
- [ ] No hard-coded colours outside the CSS custom properties file.

## 4. Demo script (DOD-9)

1. Open `/` → grid of seeded trucks.
2. Filter: ประเภทรถ = 6 ล้อ, ราคา 500,000–2,000,000, ปี ≥ 2015 → results narrow; sort by ราคา ต่ำ→สูง.
3. Click a card → detail; click thumbnails; show LINE / phone buttons.
4. Back to home → sort by "ดูมากที่สุด" → the viewed truck moves up.
5. "จัดการสินค้า" → "+ สร้างรายการใหม่" → try a duplicate title and a negative price (errors shown) → fix and save.
6. Edit the new listing: change cover image, change price → save → verify on home.
7. Delete it → confirm → gone from table and home.

## 5. AI usage record (DOD-10)

Agents append to `docs/AI-USAGE.md` after each task: what was generated, which files, and a "Human review" line left for the reviewer to fill (what was checked, what was changed and why).
