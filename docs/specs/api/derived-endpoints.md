# Derived Endpoints — Truck2Hand

> **Status: DERIVED.** These endpoints are not specified in Confluence but are required by Stories 2–3 and the DoD. They follow [conventions.md](conventions.md) and reuse the POST body and validation from [post-create-product.md](post-create-product.md). Replace this file if official specs are published (REQUIREMENTS Q1).

| # | Method | Path | Used by |
| --- | --- | --- | --- |
| D1 | GET | `/v1/api/products/:id` | Detail page; edit modal prefill |
| D2 | PUT | `/v1/api/products/:id` | Edit modal (incl. change cover) |
| D3 | DELETE | `/v1/api/products/:id` | Trash button |
| D4 | POST | `/v1/api/products/:id/views` | Detail page open (most-viewed sort) |
| D5 | POST | `/v1/api/uploads` | Image inputs in the modal |
| D6 | GET | `/v1/api/vehicle-types` | Filter + dropdown |
| D7 | GET | `/v1/api/brands` | Filter + dropdown |
| D8 | GET | `/v1/api/models?brand_id=` | Filter + dropdown (dependent on brand) |

Path param `id` must be a positive integer (else 400 `T2H1006`); unknown id → 404 `T2H1004`.

## D1 — GET product detail

Response 200:

```json
{
  "id": 1,
  "title": "Hino FC210 รถบรรทุก 6 ล้อ สภาพพร้อมใช้",
  "description": "รถใช้งานน้อย เครื่องดี เอกสารครบ",
  "price": 1480000,
  "year": 2018,
  "viewCount": 42,
  "vehicleType": { "id": 2, "name": "6 ล้อ" },
  "brand": { "id": 1, "name": "HINO" },
  "model": { "id": 1, "name": "FC210" },
  "seller": {
    "id": 1,
    "name": "คุณสมชาย รถดี",
    "phone": "0812345678",
    "lineUrl": "https://line.me/ti/p/somchai",
    "avatarUrl": "http://localhost:8080/uploads/sellers/somchai.jpg",
    "avatarPath": "/uploads/sellers/somchai.jpg",
    "isVerified": true
  },
  "images": [
    { "id": 10, "imageUrl": "http://localhost:8080/uploads/products/hino-fc210-1.jpg", "imagePath": "/uploads/products/hino-fc210-1.jpg", "isCover": true, "sortOrder": 1 },
    { "id": 11, "imageUrl": "http://localhost:8080/uploads/products/hino-fc210-2.jpg", "imagePath": "/uploads/products/hino-fc210-2.jpg", "isCover": false, "sortOrder": 2 }
  ],
  "createdAt": "2026-09-03T10:00:00+07:00",
  "updatedAt": "2026-09-03T10:00:00+07:00"
}
```

Images are ordered cover first, then by `sortOrder`. `*Path` fields let the edit modal send stored paths back in D2. D1 does **not** change `view_count` (GET stays side-effect free).

## D2 — PUT update product

- Body: identical to POST create; full replacement (not a partial patch).
- Same validation; the uniqueness check excludes the product being edited.
- In one transaction: update `products` (set `updated_at`, keep `created_at` and `view_count`), upsert seller, replace the product's `product_images` rows with the submitted list.
- Image files no longer referenced by any row are deleted from disk after commit.
- Changing which image has `isCover: true` is how the cover is changed (AC-3.13).
- Response 200: same shape as POST create.

## D3 — DELETE product

- Hard delete of the product; its `product_images` rows are removed by `ON DELETE CASCADE` **(CF: "รูปถูกลบตาม")** and their files are deleted from disk.
- The seller row is kept (it may be shared).
- Response **204** no body. Unknown id → 404.

## D4 — POST record a view

- Increments `products.view_count` by 1 atomically (`UPDATE … SET view_count = view_count + 1`).
- Response 200: `{ "id": 1, "viewCount": 43 }`. Unknown id → 404.
- No de-duplication per visitor (not required).

## D5 — POST upload image

- `multipart/form-data` via multer; fields: `file` (single file), `kind` = `product` | `seller`.
- Accept only `image/png` and `image/jpeg` with extension `.png`, `.jpg` or `.jpeg`; max 5 MB (REQUIREMENTS Q6). Otherwise 400 `T2H1006` with `field: "file"`.
- Saved to `uploads/products/` or `uploads/sellers/` with a generated unique filename (never the client filename).
- Response 201: `{ "path": "/uploads/products/9f2c…e1.jpg", "url": "http://localhost:8080/uploads/products/9f2c…e1.jpg" }`.
- Files uploaded but never attached to a product may be left on disk (cleanup is out of scope).

## D6–D8 — Lookups

```json
GET /v1/api/vehicle-types   →  [ { "id": 1, "name": "4 ล้อ" }, { "id": 2, "name": "6 ล้อ" }, … ]
GET /v1/api/brands          →  [ { "id": 1, "name": "HINO" }, … ]
GET /v1/api/models?brand_id=1 → [ { "id": 1, "name": "FC210", "brandId": 1 }, … ]
```

Plain arrays (no paging), ordered by `id` for vehicle types and by `name` for brands/models. `brand_id` optional on D8; invalid value → 400. The year dropdown is generated on the client from the range in the POST validation rules (no endpoint).

## Tests to write

AC-2.9, AC-2.10 (D4 increments; D1 does not), AC-3.11, AC-3.12 (rows and files removed), AC-3.13, AC-V.2 on update (own title allowed, another product's title rejected), AC-V.5 for D5 (wrong MIME, too large), and lookup shapes.
