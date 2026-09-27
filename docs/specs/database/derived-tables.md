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

`brands` / `models` — `HINO / FC210` must be model id 1 (Confluence example). `ASSUMPTION` starter list for the dropdowns:

| Brand | Models |
| --- | --- |
| HINO | FC210, FM1A, 500 Series |
| ISUZU | NLR, FRR, FXZ |
| MITSUBISHI FUSO | Canter, Fighter |
| VOLVO | FH |
| SCANIA | R-Series |

Demo listings (REQUIREMENTS Q8 — `ASSUMPTION` yes): 6 listings covering all four vehicle types, prices 350,000–3,200,000, years 2012–2022, at least one unverified seller and one without LINE. Images are placeholder PNGs generated locally by the seed script into `backend/uploads/` (no network downloads). Run with `npm run seed` in `backend/`; `npm run seed:reset` drops and recreates the database.
