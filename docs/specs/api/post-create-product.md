# POST Create Product

Source: Confluence [(Example) POST Create Product API](https://tripetchgroup-cdo.atlassian.net/wiki/spaces/TBW/pages/5143593233/Example+POST+Create+Product+API) (v2). Validation rules come from the DoD; details marked **(DERIVED)** are assumptions.

Creates a product from the management modal. Inserts `products`, **upserts** `sellers` from the nested seller fields, and inserts `product_images`. At least one image with `isCover = true` is required. All writes run in **one transaction** (DERIVED) — a failure leaves no partial rows.

## Request

| Method | Path |
| --- | --- |
| `POST` | `/v1/api/products` |

Path params: none. Query params: none.

### Request body

| Field | Type | Required |
| --- | --- | --- |
| title | String | Yes |
| description | String | No |
| price | Number | Yes |
| year | Number | Yes |
| vehicleTypeId | Number | Yes |
| modelId | Number | Yes |
| seller | Object | Yes |
| seller.name | String | Yes |
| seller.phone | String | Yes |
| seller.lineUrl | String | No |
| seller.avatarUrl | String | No |
| seller.isVerified | Boolean | Yes |
| images | Array | Yes (1–4) |
| images[].imageUrl | String | Yes — stored path from the upload endpoint |
| images[].isCover | Boolean | Yes |
| images[].sortOrder | Number | Yes |

Brand is **not** sent — it is derived from the model (`modelId` must belong to an existing brand). **(CF)**

## Validation rules

Implemented in the service layer and unit-tested. Every failure returns 400 `T2H1006` with one `errors[]` entry per failing field (collect all, don't stop at the first).

| Field | Rule | AC |
| --- | --- | --- |
| title | Trim; required; 1–150 chars (DERIVED length); unique among products, compared trimmed and case-insensitive | AC-V.1, AC-V.2 |
| title, description | Must not contain a prohibited keyword (case-insensitive substring, list in `backend/src/config/prohibited-keywords.js`) | AC-V.4 |
| description | Optional; trimmed; max 5,000 chars (DERIVED) | — |
| price | Integer, 1 ≤ price ≤ 99,999,999 | AC-V.3 |
| year | Integer, 1980 ≤ year ≤ current year + 1 (DERIVED range; same range feeds the dropdown) | — |
| vehicleTypeId | Must exist in `vehicle_types` | AC-V.6 |
| modelId | Must exist in `models` (brand comes from it) | AC-V.6 |
| seller.name | Trim; required; max 100 chars | — |
| seller.phone | Digits only after removing spaces/dashes; Thai format `0` + 8–9 digits (DERIVED) | AC-V.7 |
| seller.lineUrl | Optional; if present must start with `https://line.me/` (DERIVED) | AC-V.7 |
| seller.avatarUrl | Optional; must be a path previously returned by the upload endpoint under `/uploads/sellers/` | — |
| images | 1 to 4 items; exactly one `isCover: true`; each `imageUrl` under `/uploads/products/` and the file exists; `sortOrder` unique positive integers | AC-V.5, AC-3.13 |

Seller upsert (DERIVED, REQUIREMENTS Q4): look up by normalised `phone`; if found, update name / lineUrl / avatarUrl / isVerified and reuse its id; otherwise insert.

The management form collects `vehicleTypeId` from the ประเภทรถ dropdown and `seller.isVerified` from the "ผู้ขายยืนยันตัวตน" checkbox (REQUIREMENTS C7). The server does not infer either field.

Prohibited-keyword starter list (placeholder until Q2 is answered): `รถไม่มีเล่ม`, `รถสวมทะเบียน`, `รถหนีภาษี`, `รถโจร`, `ยาเสพติด`, `อาวุธ`, `ปืน`. Error message: "ไม่อนุญาตให้ลงประกาศสินค้าผิดกฎหมาย".

## Response — 201 Created (DERIVED status; CF body)

| Field | Type | Description |
| --- | --- | --- |
| id | Number | Created product id |
| title | String | |
| price | Number | |
| year | Number | |
| modelId | Number | |
| vehicleTypeId | Number | |
| sellerId | Number | Created or reused seller id |

## Notes (CF)

- Authentication: none (public `/v1/api/**`).
- `modelId` must belong to an existing brand; brand is taken from the model and not sent again.
- Errors: `T2H1006` Invalid Parameter (400), `T2H9999` System Error (500).

## Example

Paths below use the `/uploads/...` form (conflict C2; Confluence shows `/images/...`).

```json
POST /v1/api/products
{
  "title": "Hino FC210 รถบรรทุก 6 ล้อ สภาพพร้อมใช้",
  "description": "รถใช้งานน้อย เครื่องดี เอกสารครบ",
  "price": 1480000,
  "year": 2018,
  "vehicleTypeId": 2,
  "modelId": 1,
  "seller": {
    "name": "คุณสมชาย รถดี",
    "phone": "0812345678",
    "lineUrl": "https://line.me/ti/p/somchai",
    "avatarUrl": "/uploads/sellers/somchai.jpg",
    "isVerified": true
  },
  "images": [
    { "imageUrl": "/uploads/products/hino-fc210-1.jpg", "isCover": true, "sortOrder": 1 },
    { "imageUrl": "/uploads/products/hino-fc210-2.jpg", "isCover": false, "sortOrder": 2 }
  ]
}
```

```json
201 Created
{
  "id": 1,
  "title": "Hino FC210 รถบรรทุก 6 ล้อ สภาพพร้อมใช้",
  "price": 1480000,
  "year": 2018,
  "modelId": 1,
  "vehicleTypeId": 2,
  "sellerId": 1
}
```

## Tests to write

AC-3.10 (happy path, seller reused on second create with same phone), and one test per rule in the validation table: AC-V.1, AC-V.2, AC-V.3 (−1, 0, 1.5, "abc", 100000000; boundary 1 and 99,999,999 accepted), AC-V.4, AC-V.5 (0 images, 5 images, 0 covers, 2 covers), AC-V.6, AC-V.7, and a transaction test (failure after product insert leaves no rows).
