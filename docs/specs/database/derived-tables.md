# Derived Tables — Truck2Hand

> **Status: DERIVED.** The Confluence API pages name these tables as sources (`models`, `brands`, `vehicle_types`, `sellers`, `product_images`) but do not define them. Columns below are the minimum needed by the API fields. Replace this file if official specs are published (REQUIREMENTS Q1).

Conventions: same as [`products.md`](products.md) — INTEGER AUTOINCREMENT primary keys, ISO-8601 TEXT timestamps set by the app, `PRAGMA foreign_keys = ON`.

## `vehicle_types`

| Field | Required | Type | Description |
| --- | --- | --- | --- |
| id | Yes | INTEGER | PK |
| name | Yes | TEXT UNIQUE | Display name |

## `brands`

| Field | Required | Type | Description |
| --- | --- | --- | --- |
| id | Yes | INTEGER | PK |
| name | Yes | TEXT UNIQUE | e.g. `HINO` |

## `models`

| Field | Required | Type | Description |
| --- | --- | --- | --- |
| id | Yes | INTEGER | PK |
| brand_id | Yes | INTEGER | FK → `brands.id` |
| name | Yes | TEXT | e.g. `FC210`; UNIQUE (`brand_id`, `name`) |

## `sellers`

| Field | Required | Type | Description |
| --- | --- | --- | --- |
| id | Yes | INTEGER | PK |
| name | Yes | TEXT | Seller name |
| phone | Yes | TEXT UNIQUE | Normalised digits; upsert key (REQUIREMENTS Q4) |
| line_url | No | TEXT | LINE link |
| avatar_url | No | TEXT | Stored path, e.g. `/uploads/sellers/somchai.jpg` (CF: path only, no binary) |
| is_verified | Yes | INTEGER | 0/1, default 0 |
| created_at | Yes | TEXT | |
| updated_at | Yes | TEXT | |

## `product_images`

| Field | Required | Type | Description |
| --- | --- | --- | --- |
| id | Yes | INTEGER | PK |
| product_id | Yes | INTEGER | FK → `products.id` **ON DELETE CASCADE** (CF: images deleted with product) |
| image_url | Yes | TEXT | Stored path, e.g. `/uploads/products/x.jpg` (CF: path only) |
| is_cover | Yes | INTEGER | 0/1 |
| sort_order | Yes | INTEGER | 1–4 |

Constraints: UNIQUE (`product_id`, `sort_order`); partial unique index `ON product_images(product_id) WHERE is_cover = 1` guarantees at most one cover. "At least one cover" and "max 4 images" are enforced in the service layer.

## Seed data (idempotent, run at startup)

`vehicle_types` — ids fixed so the Confluence example (`vehicleTypeId: 2` = 6 ล้อ) holds:

| id | name |
| --- | --- |
| 1 | 4 ล้อ |
| 2 | 6 ล้อ |
| 3 | 10 ล้อ |
| 4 | หัวลาก |

`brands` / `models` — insert in this order so `HINO / FC210` is model id 1 (Confluence example). Names below include the homepage cards (`GXZ`, `NPR`, `WU`, `FC`) plus the original starter list. This is demo data; it does not have to match a real catalog.

| Brand | Models (insert in this order) |
| --- | --- |
| HINO | FC210, FC, FM1A, 500 Series, WU |
| ISUZU | NLR, FRR, FXZ, GXZ, NPR |
| MITSUBISHI FUSO | Canter, Fighter |
| VOLVO | FH |
| SCANIA | R-Series |

Demo listings (REQUIREMENTS Q8): 8 listings using the homepage names and the models added above. Every seller is verified and has a `https://line.me/` URL. Each of the 8 listings has one cover JPEG from the [demo listing photo catalog](demo-listing-photos.md), copied at seed time to `/uploads/products/seed-product-N.jpg`. Seller avatars stay minimal PNGs. Seed matches a listing by its canonical title. Vehicle type is the value in this table; the create form sets it from the ประเภทรถ dropdown.

| # | Title | Brand | Model | Year | Price (THB) | Seller | Vehicle type |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | หัวลาก 12ล้อ GXZ 360 JT หางแซมิเทรลเลอร์ | ISUZU | GXZ | 2021 | 2780000 | JJautocar | หัวลาก |
| 2 | ISUZU NLR130 ป66 กระบะคาร์โก้ ไม่ติดเวลา | ISUZU | NLR | 2023 | 750000 | JJautocar | 6 ล้อ |
| 3 | 6 ล้อดั๊มพ์ HINO DUTRO 12แรง | HINO | WU | 2004 | 520000 | อนะ รถบรรทุก | 6 ล้อ |
| 4 | Isuzu 6 ล้อ กระบะสแตนเลส 150 แรง เครื่องดี ยางดี | ISUZU | NPR | 2003 | 380000 | นาย Truck2Hand | 6 ล้อ |
| 5 | Hino FC210 ป65 6ล้อ ตู้เย็น ยาว6.5 เมตร | HINO | FC | 2022 | 1480000 | เต็ง ซื้อขายรถบรรทุก | 6 ล้อ |
| 6 | ISUZU NLR130 4ล้อ ไม่ติดเวลา ป66 ไมล์ 1x,xxx | ISUZU | NLR | 2023 | 899000 | เต็ง ซื้อขายรถบรรทุก | 4 ล้อ |
| 7 | 10 ล้อ HINO FM1A | HINO | FM1A | 2016 | 980000 | อนะ รถบรรทุก | 10 ล้อ |
| 8 | หัวลาก SCANIA R-Series | SCANIA | R-Series | 2019 | 3200000 | นาย Truck2Hand | หัวลาก |

Listings 7 and 8 are fillers so the count is 8; they are not visible in the homepage crop. Sellers may share a phone when the screenshot shows the same name on more than one card (JJautocar, เต็ง ซื้อขายรถบรรทุก, อนะ รถบรรทุก, นาย Truck2Hand). Run with `npm run seed` in `backend/`; `npm run seed:reset` drops and recreates the database.
