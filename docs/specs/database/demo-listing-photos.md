# Demo listing photos

Cover photos for the eight demo listings in [`derived-tables.md`](derived-tables.md). Row order is `LISTINGS` in `backend/src/db/seed.js`.

Match is best-effort. The files are already downloaded. A builder must use the local file. Do not search or download again unless a listed file is missing.

Seed copies each file to `backend/uploads/products/seed-product-N.jpg` and stores that path on the listing cover. `backend/uploads/` is gitignored. Seller avatars stay the minimal PNG from `backend/src/db/placeholder-images.js`. Runtime seed and server start must not fetch the network.

Leave the binaries in `backend/src/db/demo-photos/`. That directory is what the app and tests read.

| # | Title | Brand | Model | Local file | Stored cover path | Wikimedia Commons | Direct JPEG |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | หัวลาก 12ล้อ GXZ 360 JT หางแซมิเทรลเลอร์ | ISUZU | GXZ | `backend/src/db/demo-photos/seed-product-1.jpg` | `/uploads/products/seed-product-1.jpg` | https://commons.wikimedia.org/wiki/File:Isuzu_Plaza_Isuzu_Giga_Tractor_Custom.jpg | https://upload.wikimedia.org/wikipedia/commons/d/d1/Isuzu_Plaza_Isuzu_Giga_Tractor_Custom.jpg |
| 2 | ISUZU NLR130 ป66 กระบะคาร์โก้ ไม่ติดเวลา | ISUZU | NLR | `backend/src/db/demo-photos/seed-product-2.jpg` | `/uploads/products/seed-product-2.jpg` | https://commons.wikimedia.org/wiki/File:Isuzu_Elf_NLR_Dropside_Cargo_2019.jpg | https://upload.wikimedia.org/wikipedia/commons/a/af/Isuzu_Elf_NLR_Dropside_Cargo_2019.jpg |
| 3 | 6 ล้อดั๊มพ์ HINO DUTRO 12แรง | HINO | WU | `backend/src/db/demo-photos/seed-product-3.jpg` | `/uploads/products/seed-product-3.jpg` | https://commons.wikimedia.org/wiki/File:Hino_Dutro_X_Dump_2012.jpg | https://upload.wikimedia.org/wikipedia/commons/a/ad/Hino_Dutro_X_Dump_2012.jpg |
| 4 | Isuzu 6 ล้อ กระบะสแตนเลส 150 แรง เครื่องดี ยางดี | ISUZU | NPR | `backend/src/db/demo-photos/seed-product-4.jpg` | `/uploads/products/seed-product-4.jpg` | https://commons.wikimedia.org/wiki/File:Isuzu_Elf_NPR_Dropside_Cargo_Truck_2019.jpg | https://upload.wikimedia.org/wikipedia/commons/2/2d/Isuzu_Elf_NPR_Dropside_Cargo_Truck_2019.jpg |
| 5 | Hino FC210 ป65 6ล้อ ตู้เย็น ยาว6.5 เมตร | HINO | FC | `backend/src/db/demo-photos/seed-product-5.jpg` | `/uploads/products/seed-product-5.jpg` | https://commons.wikimedia.org/wiki/File:Across_Transport_65_Ranger_FC_Dry_van.jpg | https://upload.wikimedia.org/wikipedia/commons/9/95/Across_Transport_65_Ranger_FC_Dry_van.jpg |
| 6 | ISUZU NLR130 4ล้อ ไม่ติดเวลา ป66 ไมล์ 1x,xxx | ISUZU | NLR | `backend/src/db/demo-photos/seed-product-6.jpg` | `/uploads/products/seed-product-6.jpg` | https://commons.wikimedia.org/wiki/File:2020_Isuzu_Elf_NMR_71T_SD_L_4.6_(20210211).jpg | https://upload.wikimedia.org/wikipedia/commons/4/45/2020_Isuzu_Elf_NMR_71T_SD_L_4.6_%2820210211%29.jpg |
| 7 | 10 ล้อ HINO FM1A | HINO | FM1A | `backend/src/db/demo-photos/seed-product-7.jpg` | `/uploads/products/seed-product-7.jpg` | https://commons.wikimedia.org/wiki/File:Hino_500_FM_2826_2014_(14054088067).jpg | https://upload.wikimedia.org/wikipedia/commons/2/2e/Hino_500_FM_2826_2014_%2814054088067%29.jpg |
| 8 | หัวลาก SCANIA R-Series | SCANIA | R-Series | `backend/src/db/demo-photos/seed-product-8.jpg` | `/uploads/products/seed-product-8.jpg` | https://commons.wikimedia.org/wiki/File:Scania_R420_at_Scania_AB.jpg | https://upload.wikimedia.org/wikipedia/commons/b/bb/Scania_R420_at_Scania_AB.jpg |
