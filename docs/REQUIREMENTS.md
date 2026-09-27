# Requirements: Truck2Hand

- **Application name:** Truck2Hand — a used-truck marketplace (storefront + back-office)
- **UI language:** Thai. Use the exact Thai UI copy in [§ UI copy](#ui-copy); do not translate or paraphrase it.

This file is the product source of truth for `app-architect` and `app-builder`. **Do not add features that are not listed here or in the linked spec files.** When something is missing or ambiguous, follow the marked `ASSUMPTION` and record it in the build notes; do not invent new behaviour.

## Context files (read before designing or coding)

| File | What it defines | Origin |
| --- | --- | --- |
| [`docs/specs/acceptance-criteria.md`](docs/specs/acceptance-criteria.md) | Testable acceptance criteria per story (AC IDs) | Confluence: Acceptance Criteria |
| [`docs/specs/definition-of-done.md`](docs/specs/definition-of-done.md) | Pinned stack, quality gates, demo script | Confluence: Definition of Done |
| [`docs/specs/api/conventions.md`](docs/specs/api/conventions.md) | Base URL, errors, image URLs, Swagger | Confluence: API pages (shared notes) |
| [`docs/specs/api/get-product-list.md`](docs/specs/api/get-product-list.md) | `GET /v1/api/products` | Confluence: (Example) GET Product List API |
| [`docs/specs/api/post-create-product.md`](docs/specs/api/post-create-product.md) | `POST /v1/api/products` | Confluence: (Example) POST Create Product API |
| [`docs/specs/api/derived-endpoints.md`](docs/specs/api/derived-endpoints.md) | Detail, update, delete, upload, views, lookups | **DERIVED** — needed by the stories, not in Confluence |
| [`docs/specs/database/products.md`](docs/specs/database/products.md) | `products` table | Confluence: (Example) products |
| [`docs/specs/database/derived-tables.md`](docs/specs/database/derived-tables.md) | `brands`, `models`, `vehicle_types`, `sellers`, `product_images` + seed data | **DERIVED** — referenced by the API pages, not specified in Confluence |

## UI examples

Use these screenshots as a visual guide. They are not a second product spec. When a screenshot disagrees with this file or the acceptance criteria, follow the written spec.

- Home page: docs/specs/ui/homepage.png
- Manage listing: docs/specs/ui/manage_listing.png
- Listing details: docs/specs/ui/listing_details.png
- Create a new listing: docs/specs/ui/create_a_new_listing.png
- Confirmation to delete a listing: docs/specs/ui/confirmation_to_delete_a_listing.png

### Precedence when files disagree

1. `definition-of-done.md` (release gate)
2. `acceptance-criteria.md`
3. Confluence-origin API / database specs
4. This file (product narrative, look and feel)
5. `DERIVED` specs — lowest; they fill gaps only

Known conflicts are already resolved inside the files and listed in [§ Resolved conflicts](#resolved-conflicts).

## Stack (summary — exact versions in `definition-of-done.md`)

- **Monorepo** with two packages: `frontend/` and `backend/`. Runtime **Node.js 24.20.0** (add `.nvmrc`).
- **Frontend:** Vue 3 + vue-router + Vite. Plain CSS with custom properties. No UI kit, no CSS framework, no icon pack, no HTTP client library (use `fetch`).
- **Backend:** Express 5 + Drizzle ORM on better-sqlite3 (SQLite file `backend/data/truck2hand.db`), multer for image upload, cors, swagger-ui-express at `/api-docs`.
- **Ports:** backend `http://localhost:8080` (from the API spec), frontend Vite dev server `http://localhost:5173`.
- **Dependencies are pinned exactly** (no `^`/`~`). No runtime dependency beyond the DoD list. `ASSUMPTION`: JavaScript (ES modules), not TypeScript, because TypeScript tooling is not in the pinned list.
- **Backend layering:** `routes/` (HTTP only) → `services/` (validation + business rules, unit-tested) → `repositories/` (Drizzle queries). Keep business rules out of route handlers so they can be tested without HTTP.
- **Tests:** backend unit + API tests with the built-in **`node:test`** runner and `node:assert` (no extra dependency; HTTP tests start the app on an ephemeral port and use global `fetch`). Tests use a fresh in-memory or temp-file SQLite database per test file. `npm test` in `backend/` is the pass/fail source. No Playwright / E2E as pass/fail source.

## Product

Three pages that link to each other (routes are `ASSUMPTION`s):

| Page | Route | Story |
| --- | --- | --- |
| Home — product grid with filters and sort | `/` | Story 1 |
| Product detail | `/products/:id` | Story 2 |
| Product management (back-office) | `/manage` | Story 3 |

No authentication (the API is public per the Confluence notes). The back-office is reachable from the header button.

### Story 1 — Home (product grid)

- **Header** on every storefront page: logo text "Truck2Hand" top-left (links to `/`), button "จัดการสินค้า" top-right (goes to `/manage`).
- **Grid** of product cards, 3–4 per row on desktop (see responsive rules).
- **Card**, top to bottom: large cover image → title → gray small meta line `ยี่ห้อ | รุ่น | ปี` → price in large red text → seller avatar + seller name + green badge "ผู้ขายยืนยันตัวตน" (shown only when `sellerIsVerified` is true) → bottom-right: LINE icon button and phone icon button.
  - Clicking the card (image or title) opens `/products/:id`.
  - Phone button → `tel:{sellerPhone}`. The LINE icon button is always shown, including when `sellerLineUrl` is null. Clicking it shows the message "ติดต่อผู้ขายรายนี้ผ่าน LINE" and does not open a URL. Clicking these buttons must not also navigate to detail.
- **Filters** (all combinable at once, applied together): one vehicle type (4 ล้อ, 6 ล้อ, 10 ล้อ, หัวลาก — single select, query `vehicle_type_id`), price range (min/max), year range labeled "ปีที่ผลิต" (min/max), brand, and model. Model is a dropdown of models for the selected brand and stays disabled until a brand is selected. A "ล้างตัวกรอง" control resets all filters.
- **Sort** dropdown, four behaviors only: newest (default), price low→high, price high→low, most viewed. Demo script labels for two of them are "ราคา ต่ำ→สูง" and "ดูมากที่สุด". No other sort options.
- Filter and sort state is kept in the URL query string so reload and back-navigation keep it.
- **Pagination:** `ASSUMPTION` simple previous/next pager with "หน้า X / Y"; request `per_page=12` (fills 3- and 4-column rows).
- **Empty state** when no products match: "ไม่พบสินค้าที่ตรงกับเงื่อนไข".
- **Price format:** Thai baht with thousands separators, e.g. `฿1,480,000`.
- The `q` title search exists in the API but there is **no search box** in the UI (not in the stories).

### Story 2 — Product detail

- Two columns: left gallery, right information.
- **Left:** large main image on top; thumbnails in a row below; clicking a thumbnail swaps the main image. The cover image is shown first.
- **Right:** title and price (large red) at top → full description (preserve line breaks) → vehicle box (ยี่ห้อ, รุ่น, ปีที่ผลิต) → seller box (ชื่อ, เบอร์โทรศัพท์, ลิงก์ Line) → large buttons at the bottom: "ติดต่อผู้ขาย" (`tel:` link) and "แอดไลน์".
  - Do not add the screenshot's breadcrumb or "โพสต์เมื่อ".
  - "แอดไลน์" is always shown. Clicking it shows "ติดต่อผู้ขายรายนี้ผ่าน LINE" and does not open a URL.
- When the page opens, record one view (`POST /v1/api/products/:id/views`) — this feeds the "most viewed" sort.
- Unknown id → "ไม่พบสินค้านี้" with a link back to home.

### Story 3 — Product management (back-office)

- Heading "ระบบจัดการสินค้า"; a link back to the storefront.
- Primary button "+ สร้างรายการใหม่" opens the create modal.
- **Table** columns: รูปภาพ | ชื่อสินค้า | ยี่ห้อ | ราคา | จัดการ. Row: cover thumbnail, title, brand, price, pencil button (edit → opens the same modal prefilled) and trash button (delete → confirm dialog "ยืนยันการลบสินค้านี้?" → hard delete). Sorted newest first, same pager as home.
- **Modal form (create / edit):**
  - Product images: PNG/JPG only, add and remove, **1–4 images**, exactly one marked as cover ("ตั้งเป็นภาพปก"); the first uploaded image is the cover by default; the cover can be changed when editing (DoD).
  - Text inputs: ชื่อรถ, ราคา; textarea: รายละเอียด.
  - Dropdowns: ประเภทรถ, ยี่ห้อ, รุ่น (filtered by brand), ปีที่ผลิต. Ignore the create screenshot, which omits ประเภทรถ and labels the year "ปีรถ".
  - Seller: avatar upload (PNG/JPG, optional), ชื่อคนขาย, เบอร์โทร, ลิงก์ไลน์ (optional), and checkbox "ผู้ขายยืนยันตัวตน". Ignore the screenshot's required LINE mark and its missing checkbox.
  - Footer buttons: "ยกเลิก" (close without saving) and "บันทึกข้อมูล" (save).
  - Field errors appear under each field; server errors (e.g. duplicate title) are mapped to the same place. Save is disabled while the request is in flight.

### Validation and error handling (from DoD)

Enforced in the backend service layer (source of truth) and mirrored in the form for fast feedback. Full rules in [`api/post-create-product.md`](docs/specs/api/post-create-product.md#validation-rules).

- Title required, trimmed, **unique** (case-insensitive, after trim) across listings.
- Price is an integer from **1 to 99,999,999** THB; negative, zero, decimal or larger values are rejected.
- **Prohibited listings:** title and description are checked against a configurable blocklist of Thai/English keywords (`backend/src/config/prohibited-keywords.js`); a match rejects the listing. See open question Q2.
- 1–4 product images, exactly one cover, PNG/JPG only.
- Every API error uses the error envelope in [`api/conventions.md`](docs/specs/api/conventions.md#errors); the UI never shows a raw stack trace.

## Look and feel

Professional and clean marketplace, not a flashy marketing site.

- **Theme:** one light theme only. Neutral page background, white card/surface, dark text.
- **Colour tokens** (CSS custom properties on `:root`, used everywhere — no hard-coded colours in components):

| Token | Suggested value | Use |
| --- | --- | --- |
| `--color-bg` | `#f8fafc` | page background |
| `--color-surface` | `#ffffff` | cards, table, modal |
| `--color-text` | `#0f172a` | main text |
| `--color-muted` | `#64748b` | meta line, secondary text |
| `--color-border` | `#e2e8f0` | borders, dividers |
| `--color-primary` | `#2563eb` (hover `#1d4ed8`) | "จัดการสินค้า", "+ สร้างรายการใหม่", "บันทึกข้อมูล", "ติดต่อผู้ขาย" |
| `--color-price` | `#dc2626` | price text |
| `--color-verified` | `#16a34a` | "ผู้ขายยืนยันตัวตน" badge |
| `--color-line` | `#06c755` | LINE buttons |
| `--color-danger` | `#dc2626` | delete — quiet style (red icon/outline, not a big red block) |

- **Type:** system font stack with Thai fallbacks: `system-ui, -apple-system, "Segoe UI", "Leelawadee UI", "Thonburi", sans-serif`. No web fonts from the network. Hierarchy: page title > card/detail title > body > muted meta.
- **Layout:** centered container, max width ~75rem, comfortable spacing. Cards have a light shadow at most; image area uses a fixed aspect ratio (4:3) with `object-fit: cover`.
- **Icons:** LINE, phone, pencil, trash as small inline SVG components written in the repo. No icon library.
- **Responsive:** grid 4 columns ≥1280px, 3 ≥1024px, 2 ≥640px, 1 below. Detail page stacks gallery above info below 768px. Management table scrolls horizontally inside its own container on small screens; the modal becomes full-screen below 640px.
- **Motion:** none required. No animation libraries, loaders beyond a simple text "กำลังโหลด…", no transitions longer than ~150ms.
- **Do not:** dark mode, gradients, glassmorphism, heavy shadows, carousels with auto-play, extra fonts.

## UI copy

Use these strings verbatim: `Truck2Hand`, `จัดการสินค้า`, `ผู้ขายยืนยันตัวตน`, `ติดต่อผู้ขาย`, `แอดไลน์`, `ติดต่อผู้ขายรายนี้ผ่าน LINE`, `ระบบจัดการสินค้า`, `+ สร้างรายการใหม่`, `รูปภาพ`, `ชื่อสินค้า`, `ยี่ห้อ`, `รุ่น`, `ปี`, `ปีที่ผลิต`, `ราคา`, `จัดการ`, `ยกเลิก`, `บันทึกข้อมูล`, `ประเภทรถ`, `4 ล้อ`, `6 ล้อ`, `10 ล้อ`, `หัวลาก`, `ล้างตัวกรอง`. The year field is labeled `ปีที่ผลิต` on the filter, the detail page, and the create/edit dropdown. The card meta line stays the short pattern `ยี่ห้อ | รุ่น | ปี`. Strings marked `ASSUMPTION` above (empty states, confirm dialog, "ตั้งเป็นภาพปก") may be adjusted by the product owner.

## Resolved conflicts

| # | Conflict | Resolution |
| --- | --- | --- |
| C1 | DoD requires sort by most viewed; GET list `sort_by` allows only `id, price, year, created_at` | Add `view_count` to `sort_by` and a `view_count` column (derived). |
| C2 | POST example uses image paths `/images/products/…`; GET notes say files live in `uploads/` served at `/uploads/**` | Store paths as `/uploads/products/<file>` and `/uploads/sellers/<file>`; the POST example is treated as illustrative. |
| C3 | Detail, filter, and form used "ปีที่ผลิต", "ปีจดทะเบียน", and "ปีรถ" for one `year` column | One field `year`. Every user-facing label is "ปีที่ผลิต". |
| C4 | `products` table has no foreign keys, but the POST body carries `vehicleTypeId`, `modelId` and a seller | Add `vehicle_type_id`, `model_id`, `seller_id`, `view_count` (see `database/products.md` § Derived columns). |
| C5 | Only two endpoints specified; Stories 2–3 need detail, update, delete, upload and dropdown data | Specified in `api/derived-endpoints.md`, reusing the same conventions. |
| C6 | Screenshots show search, bookmarks, multi-select vehicle type, a type-in model field, manage สถานะ, and different headings | Do not build those. Search UI and favourites stay out of scope. Filters, sort, and manage wording follow this file. See the screenshot table below. |
| C7 | Create mock has no vehicle-type dropdown and no verified checkbox, and marks LINE required | Follow this file: show ประเภทรถ and "ผู้ขายยืนยันตัวตน". `seller.lineUrl` stays optional. Year label is "ปีที่ผลิต", not the mock's "ปีรถ". |
| C8 | LINE button opened `sellerLineUrl`; acceptance criteria always showed it | Always show the LINE button. Click shows "ติดต่อผู้ขายรายนี้ผ่าน LINE" and does not open a URL. |

### Screenshot differences that this file does not adopt

| Screenshot | What it shows | What to build |
| --- | --- | --- |
| Home | Header search, card bookmark, title "รายการรถบรรทุกมือสองล่าสุด", "พบสินค้าทั้งหมด N รายการ", vehicle-type checkboxes, type-in model, clear label "ล้างค่า", no sort dropdown | No search box, no bookmark, no result-count title. One vehicle-type select, brand-dependent model dropdown, "ล้างตัวกรอง", and the four sort behaviors. |
| Manage | "ระบบจัดการหลังบ้าน", "จัดการรายการรถบรรทุก", "+ เพิ่มรายการใหม่", columns ปี and สถานะ ("กำลังขาย"), pager "แสดง 1 ถึง 2 จาก 2 รายการ" | Story 3 wording and columns. No status column in the database. |
| Delete dialog | "ยืนยันการลบรายการ" and a solid red "ลบรายการ" button | Confirm "ยืนยันการลบสินค้านี้?". Delete stays a quiet red control. |
| Detail | "ปีรถ", "โพสต์เมื่อ", buttons "แชทผ่าน LINE" / "โทรติดต่อ" | Vehicle box uses "ปีที่ผลิต". Seller box shows name, phone, and LINE link. Buttons are "ติดต่อผู้ขาย" and "แอดไลน์". No breadcrumb and no "โพสต์เมื่อ". |
| Create | No ประเภทรถ, no verified checkbox, LINE marked required, year label "ปีรถ" | Show ประเภทรถ and "ผู้ขายยืนยันตัวตน". LINE is optional. Year dropdown label is "ปีที่ผลิต". |

## Open questions (confirm with product owner)

Each has a working `ASSUMPTION` so the build is not blocked.

- **Q1** Are the `DERIVED` endpoints and tables acceptable, or will Confluence pages be written for them?
- **Q2** What counts as "สินค้าผิดกฎหมาย"? Current assumption: keyword blocklist (e.g. `รถไม่มีเล่ม`, `รถสวมทะเบียน`, `รถหนีภาษี`, `ยาเสพติด`, `อาวุธ`). Supply the real list or a different rule.
- **Q3** Resolved: "ผู้ขายยืนยันตัวตน" is the create/edit checkbox and is sent as `seller.isVerified`.
- **Q4** Seller upsert key — assumed `phone` (unique). Same phone with a different name updates the seller's name/LINE/avatar.
- **Q5** Error code and HTTP status for a duplicate title — assumed `T2H1006` / 400 with a field error (only two codes exist in Confluence).
- **Q6** Max image size — assumed 5 MB per file.
- **Q7** Test runner — assumed `node:test` (no extra dependency). Is Vitest/Jest + supertest allowed as a dev dependency instead?
- **Q8** Resolved: seed lookup data plus the demo listings in `database/derived-tables.md` (homepage names, including models that were missing from the starter list).

## Out of scope

Authentication/roles, payments, chat, favourites, seller accounts, search box UI, multiple languages, dark mode, soft delete/undo, analytics beyond the view counter, stg/prd deployment (base URLs are TBD), web fonts, animation libraries.

## Sources

Confluence space TBW (AMATERASU), retrieved 2026-09-27: [Requirement](https://tripetchgroup-cdo.atlassian.net/wiki/spaces/TBW/pages/5141364961/Requirement) (v3), [Acceptance Criteria](https://tripetchgroup-cdo.atlassian.net/wiki/spaces/TBW/pages/5142872157/Acceptance+Criteria) (v1), [Definition of Done](https://tripetchgroup-cdo.atlassian.net/wiki/spaces/TBW/pages/5143265326/Definition+of+Done) (v12), [POST Create Product API](https://tripetchgroup-cdo.atlassian.net/wiki/spaces/TBW/pages/5143593233/Example+POST+Create+Product+API) (v2), [GET Product List API](https://tripetchgroup-cdo.atlassian.net/wiki/spaces/TBW/pages/5143298279/Example+GET+Product+List+API) (v1), [products table](https://tripetchgroup-cdo.atlassian.net/wiki/spaces/TBW/pages/5143593246/Example+products) (v3).
