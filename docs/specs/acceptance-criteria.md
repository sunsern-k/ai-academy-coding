# Acceptance Criteria — Truck2Hand

Source: Confluence [Acceptance Criteria](https://tripetchgroup-cdo.atlassian.net/wiki/spaces/TBW/pages/5142872157/Acceptance+Criteria) (v1), plus DoD-driven criteria marked **(DoD)**. Each criterion has an ID; tests and build notes should reference these IDs.

**Verified by:** `unit` = backend `node:test`; `api` = backend HTTP test; `manual` = checked in the demo (frontend has no automated pass/fail gate).

## Story 1 — Home (product grid)

| ID | Criterion | Verified by |
| --- | --- | --- |
| AC-1.1 | Header shows logo "Truck2Hand" top-left and button "จัดการสินค้า" top-right; the button navigates to `/manage`. | manual |
| AC-1.2 | Products render as a grid with 3–4 cards per row on desktop widths (≥1024px). | manual |
| AC-1.3 | Each card shows, top to bottom: large cover image; title; small gray text `ยี่ห้อ \| รุ่น \| ปี`; price in large red text; seller avatar, seller name and green tag "ผู้ขายยืนยันตัวตน" (only when verified). | manual |
| AC-1.4 | Bottom-right of each card has a LINE icon button and a phone icon button that open the seller's LINE URL and `tel:` link. | manual |
| AC-1.5 | User can filter by vehicle type (4 ล้อ, 6 ล้อ, 10 ล้อ, หัวลาก). | api + manual |
| AC-1.6 | User can filter by price range (min and/or max, inclusive). | api + manual |
| AC-1.7 | User can filter by registration-year range (min and/or max, inclusive). | api + manual |
| AC-1.8 | User can filter by brand and by model. | api + manual |
| AC-1.9 **(DoD)** | All filters can be combined in one request; results satisfy every active filter (AND). | api + manual |
| AC-1.10 **(DoD)** | User can sort by newest, price (asc/desc) and most viewed. | api + manual |
| AC-1.11 | List response follows the paging shape `size, totalPages, number, totalRecords, records` with 0-based `page`. | api |
| AC-1.12 | `coverImageUrl` and `sellerAvatarUrl` are absolute URLs usable directly in `<img src>`. | api |
| AC-1.13 | Invalid query params (e.g. `min_price=abc`, `sort_by=foo`, `order_by=UP`, `min_price > max_price`) return 400 `T2H1006`. | api |
| AC-1.14 | Empty result renders the empty state, not an error. | api + manual |

## Story 2 — Product detail

| ID | Criterion | Verified by |
| --- | --- | --- |
| AC-2.1 | Clicking a card on the home page navigates to that product's detail page. | manual |
| AC-2.2 | Layout is split clearly into left and right columns (stacks on narrow screens). | manual |
| AC-2.3 | Left: largest image on top, thumbnails in a row below; clicking a thumbnail changes the large image. | manual |
| AC-2.4 | Right top: title and price (large red text). | manual |
| AC-2.5 | Full description is displayed (line breaks preserved). | manual |
| AC-2.6 | Vehicle box shows ยี่ห้อ, รุ่น, ปีที่ผลิต. | api + manual |
| AC-2.7 | Seller box shows name, phone and LINE link. | api + manual |
| AC-2.8 | Large buttons "ติดต่อผู้ขาย" and "แอดไลน์" at the bottom. | manual |
| AC-2.9 | `GET /v1/api/products/:id` returns 404 `T2H1004` for an unknown id; the UI shows "ไม่พบสินค้านี้". | api + manual |
| AC-2.10 **(DoD)** | Opening the detail page increments the product's view count by 1. | api |

## Story 3 — Product management (back-office)

| ID | Criterion | Verified by |
| --- | --- | --- |
| AC-3.1 | Heading "ระบบจัดการสินค้า" is shown at the top. | manual |
| AC-3.2 | Button "+ สร้างรายการใหม่" opens a modal. | manual |
| AC-3.3 | Table header: รูปภาพ \| ชื่อสินค้า \| ยี่ห้อ \| ราคา \| จัดการ. | manual |
| AC-3.4 | Each row shows data matching the header, with a pencil (edit) and trash (delete) icon in "จัดการ". | manual |
| AC-3.5 | Form accepts product images (.png, .jpg) — add and remove, maximum 4. | unit + api + manual |
| AC-3.6 | Form has text inputs/textarea for ชื่อรถ, รายละเอียด, ราคา, ชื่อคนขาย, เบอร์โทร, ลิงก์ไลน์. | manual |
| AC-3.7 | Form has a seller profile image upload (.png, .jpg). | api + manual |
| AC-3.8 | Form has dropdowns for ยี่ห้อ, รุ่น, ปีรถ (and ประเภทรถ, see REQUIREMENTS). | manual |
| AC-3.9 | "ยกเลิก" closes the modal without saving; "บันทึกข้อมูล" saves to the system. | api + manual |
| AC-3.10 | Create: valid body → 201 with the created record; the product appears on home and in the table. | api |
| AC-3.11 | Edit: valid body → 200; changes appear on home, detail and table. | api |
| AC-3.12 | Delete: after confirm → product and its images are removed (hard delete); detail returns 404. | api |
| AC-3.13 **(DoD)** | Cover image of a listing can be changed on edit; exactly one image is cover. | unit + api |

## Validation and error handling (DoD)

| ID | Criterion | Verified by |
| --- | --- | --- |
| AC-V.1 | Title required; whitespace-only rejected; stored trimmed. | unit + api |
| AC-V.2 | Duplicate title (case-insensitive, trimmed) rejected on create and on edit (editing a product with its own title is allowed). | unit + api |
| AC-V.3 | Price negative, zero, decimal, non-numeric or > 99,999,999 rejected. | unit + api |
| AC-V.4 | Title or description containing a prohibited keyword rejected. | unit + api |
| AC-V.5 | 0 or more than 4 images, zero or multiple covers, or non-PNG/JPG upload rejected. | unit + api |
| AC-V.6 | Unknown `vehicleTypeId` / `modelId` rejected. | unit + api |
| AC-V.7 | Invalid phone or LINE URL format rejected. | unit + api |
| AC-V.8 | All errors use the error envelope; unexpected errors return 500 `T2H9999` without leaking stack traces. | api |
| AC-V.9 | Form shows field-level messages for both client-side and server-side validation errors. | manual |
