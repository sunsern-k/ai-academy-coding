# Table: `products`

Source: Confluence [(Example) products](https://tripetchgroup-cdo.atlassian.net/wiki/spaces/TBW/pages/5143593246/Example+products) (v3).

## Description (CF)

Truck listings for the storefront (card, filter, detail) and the back-office CRUD. Brand is taken from `models.brand_id` and is **not** stored on this table. Deletes are **hard deletes**; the product's images are deleted with it.

Database: SQLite via better-sqlite3, file `backend/data/truck2hand.db`. Enable `PRAGMA foreign_keys = ON` on every connection (SQLite has foreign keys off by default).

## Fields (CF)

| Field Name | Required | Data Type | Description | Example |
| --- | --- | --- | --- | --- |
| id | Yes | INTEGER | Primary key, AUTOINCREMENT | 1 |
| title | Yes | TEXT | Post title / truck name | Hino FC210 รถบรรทุก 6 ล้อ สภาพพร้อมใช้ |
| description | No | TEXT | Truck description (paragraph on detail page) | รถใช้งานน้อย เครื่องดี เอกสารครบ |
| price | Yes | INTEGER | Selling price, whole THB (no satang) | 1480000 |
| year | Yes | INTEGER | Truck year / registration year — used by the dropdown and year-range filter | 2018 |
| created_at | Yes | TEXT | Creation timestamp ISO-8601 (not updatable) | 2026-09-03T10:00:00+07:00 |
| updated_at | Yes | TEXT | Last update timestamp ISO-8601 (set by the app; no ON UPDATE) | 2026-09-03T10:00:00+07:00 |

## Derived columns (DERIVED — required by the API and DoD, see REQUIREMENTS conflicts C1, C4)

| Field Name | Required | Data Type | Description | Example |
| --- | --- | --- | --- | --- |
| vehicle_type_id | Yes | INTEGER | FK → `vehicle_types.id` | 2 |
| model_id | Yes | INTEGER | FK → `models.id` (brand via `models.brand_id`) | 1 |
| seller_id | Yes | INTEGER | FK → `sellers.id` | 1 |
| view_count | Yes | INTEGER | Detail-page views, default 0 — "most viewed" sort | 42 |

## Constraints and indexes (DERIVED)

```sql
CREATE TABLE IF NOT EXISTS products (
  id               INTEGER PRIMARY KEY AUTOINCREMENT,
  title            TEXT    NOT NULL,
  description      TEXT,
  price            INTEGER NOT NULL CHECK (price BETWEEN 1 AND 99999999),
  year             INTEGER NOT NULL,
  vehicle_type_id  INTEGER NOT NULL REFERENCES vehicle_types(id),
  model_id         INTEGER NOT NULL REFERENCES models(id),
  seller_id        INTEGER NOT NULL REFERENCES sellers(id),
  view_count       INTEGER NOT NULL DEFAULT 0,
  created_at       TEXT    NOT NULL,
  updated_at       TEXT    NOT NULL
);
CREATE UNIQUE INDEX IF NOT EXISTS ux_products_title_nocase ON products (trim(title) COLLATE NOCASE);
CREATE INDEX IF NOT EXISTS ix_products_filters ON products (vehicle_type_id, model_id, price, year);
CREATE INDEX IF NOT EXISTS ix_products_created ON products (created_at);
CREATE INDEX IF NOT EXISTS ix_products_views   ON products (view_count);
```

- The unique index is a safety net for AC-V.2; the service layer checks first so it can return a friendly field error. SQLite's `NOCASE` only folds ASCII letters, which is fine for Thai (no case).
- The `CHECK` on price mirrors DoD validation.
- Timestamps are written in `+07:00` (Asia/Bangkok) by the application.
