# API Conventions — Truck2Hand

Shared rules for every endpoint. Items from Confluence are marked **(CF)**; the rest are **DERIVED** to keep endpoints consistent.

## Base endpoint (CF)

| Environment | URL |
| --- | --- |
| dev | `http://localhost:8080` |
| stg | TBD |
| prd | TBD |

Read the base URL from env var `PUBLIC_BASE_URL` (default `http://localhost:8080`) so absolute image URLs follow the environment.

## General

- All API paths are under `/v1/api/`. **(CF)**
- Authentication: none — `/v1/api/**` is public. **(CF)**
- JSON request/response bodies use **camelCase**; query params use **snake_case** (as in the GET list spec). **(CF)**
- Timestamps are ISO-8601 strings with offset, e.g. `2026-09-03T10:00:00+07:00`. **(CF)**
- Money is an integer number of THB (no satang). **(CF)**
- CORS allows the frontend origin (`http://localhost:5173` in dev) via the `cors` package.
- Swagger UI at `GET /api-docs` (swagger-ui-express) from a hand-written OpenAPI 3 document in `backend/src/openapi.js`; it must describe every endpoint in this folder.

## Errors

Error codes from Confluence **(CF)**: `T2H1006` Invalid Parameter (400), `T2H9999` System Error (500).
Additional codes **(DERIVED)** needed by the derived endpoints:

| Code | HTTP | Meaning |
| --- | --- | --- |
| `T2H1004` | 404 | Resource not found |
| `T2H1006` | 400 | Invalid parameter / validation failed (includes duplicate title — see REQUIREMENTS Q5) |
| `T2H9999` | 500 | System error |

Error envelope **(DERIVED)**:

```json
{
  "code": "T2H1006",
  "message": "Invalid Parameter",
  "errors": [
    { "field": "price", "message": "ราคาต้องอยู่ระหว่าง 1 ถึง 99,999,999" },
    { "field": "title", "message": "ชื่อโพสต์นี้มีอยู่แล้ว" }
  ]
}
```

- `errors` is present for validation failures; `field` uses the request body path (e.g. `seller.phone`, `images[1].isCover`) or the query-param name.
- 500 responses never include stack traces or SQL text; log them server-side.
- Express 5 forwards rejected promises from async handlers to the error middleware, so a single error-handling middleware produces every envelope.

## Images and files (CF)

- SQLite stores only paths in `product_images.image_url` and `sellers.avatar_url` — never binary data.
- Files live in `backend/uploads/` (`uploads/products/`, `uploads/sellers/`); `GET /uploads/**` is served statically from disk.
- Stored path format **(DERIVED, resolves conflict C2)**: `/uploads/products/<generated-name>.<ext>`.
- Responses convert stored paths to absolute URLs: `PUBLIC_BASE_URL + path`, e.g. `http://localhost:8080/uploads/products/hino-fc210-1.jpg`. The frontend uses these values directly in `<img src>` and never builds paths itself.
- Request bodies send the **stored path** (as returned by the upload endpoint), not the absolute URL. The server also accepts an absolute URL with its own base and strips it back to the path.

## Pagination (CF — from GET list)

Query `page` (0-based, default 0), `per_page` (default 10, **DERIVED** max 50). Response wrapper:

```json
{ "size": 10, "totalPages": 1, "number": 0, "totalRecords": 1, "records": [] }
```

`size` = page size requested, `number` = current page index.
