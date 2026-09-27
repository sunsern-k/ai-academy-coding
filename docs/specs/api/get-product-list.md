# GET Product List

Source: Confluence [(Example) GET Product List API](https://tripetchgroup-cdo.atlassian.net/wiki/spaces/TBW/pages/5143298279/Example+GET+Product+List+API) (v1). Changes vs. Confluence are marked **(DERIVED)** with the reason.

Returns paginated product cards for the home grid and the management table. Supports filtering by vehicle type, brand, model, price range and registration-year range. Source tables: `products`, `models`, `brands`, `vehicle_types`, `sellers`, `product_images`.

## Request

| Method | Path |
| --- | --- |
| `GET` | `/v1/api/products` |

Path params: none. Request body: none.

### Query params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| page | Number | Page index (0-based) | 0 |
| per_page | Number | Records per page (**DERIVED** max 50) | 10 |
| sort_by | String | `id`, `price`, `year`, `created_at`, **`view_count` (DERIVED — DoD "most clicked")** | `created_at` |
| order_by | String | `ASC` or `DESC` | `DESC` |
| q | String | Wildcard search on title (case-insensitive substring) | - |
| vehicle_type_id | Number | Filter vehicle type | - |
| brand_id | Number | Filter brand (via `models.brand_id`) | - |
| model_id | Number | Filter model | - |
| min_price | Number | Minimum price THB (inclusive) | - |
| max_price | Number | Maximum price THB (inclusive) | - |
| min_year | Number | Minimum registration year (inclusive) | - |
| max_year | Number | Maximum registration year (inclusive) | - |

Rules:
- All filters combine with **AND** (DoD: multiple filters at once).
- Ties are broken by `id DESC` so paging is stable.
- 400 `T2H1006` when: a numeric param is not a non-negative integer; `sort_by`/`order_by` not in the allowed set; `min_price > max_price`; `min_year > max_year`; `per_page` < 1 or > 50.
- Unknown-but-valid ids (e.g. `brand_id=999`) return an empty page, not an error.
- `page` beyond the last page returns an empty `records` array with correct totals.

UI mapping of sort options: newest → `created_at DESC`; price low→high → `price ASC`; price high→low → `price DESC`; most viewed → `view_count DESC`.

## Response

| Field | Type |
| --- | --- |
| size | Number |
| totalPages | Number |
| number | Number |
| totalRecords | Long |
| records | Array |

### Record object

| Field | Type | Description |
| --- | --- | --- |
| id | Number | Product id |
| title | String | Post title / truck name |
| price | Number | Price THB |
| year | Number | Truck year / registration year |
| brandName | String | Brand |
| modelName | String | Model |
| vehicleTypeName | String | Vehicle type |
| coverImageUrl | String | Absolute URL of the cover image for `<img src>` |
| sellerName | String | Seller name |
| sellerAvatarUrl | String | Absolute URL of the seller avatar (nullable) |
| sellerIsVerified | Boolean | "ผู้ขายยืนยันตัวตน" badge |
| sellerPhone | String | Phone (call button) |
| sellerLineUrl | String | LINE link (nullable) |

## Notes (CF)

- Authentication: none (public `/v1/api/**`).
- SQLite stores only paths; files are in `uploads/`, served at `GET /uploads/**`; the API returns absolute URLs. See [conventions](conventions.md#images-and-files-cf).
- Errors: `T2H1006` Invalid Parameter (400), `T2H9999` System Error (500).

## Example (CF)

```
GET /v1/api/products?page=0&per_page=10&sort_by=created_at&order_by=DESC&vehicle_type_id=2&min_price=500000&max_year=2020
```

```json
{
  "size": 10,
  "totalPages": 1,
  "number": 0,
  "totalRecords": 1,
  "records": [
    {
      "id": 1,
      "title": "Hino FC210 รถบรรทุก 6 ล้อ สภาพพร้อมใช้",
      "price": 1480000,
      "year": 2018,
      "brandName": "HINO",
      "modelName": "FC210",
      "vehicleTypeName": "6 ล้อ",
      "coverImageUrl": "http://localhost:8080/uploads/products/hino-fc210-1.jpg",
      "sellerName": "คุณสมชาย รถดี",
      "sellerAvatarUrl": "http://localhost:8080/uploads/sellers/somchai.jpg",
      "sellerIsVerified": true,
      "sellerPhone": "0812345678",
      "sellerLineUrl": "https://line.me/ti/p/somchai"
    }
  ]
}
```

## Tests to write

AC-1.5 … AC-1.9 (each filter alone and all combined), AC-1.10 (each sort), AC-1.11 (paging shape, last page, beyond last page), AC-1.12 (absolute URLs), AC-1.13 (each invalid-param case), AC-1.14 (empty result).
