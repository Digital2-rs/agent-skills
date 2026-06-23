# Promobox Variant / Product (single record)

`GET {base}/{lang}/api/{variant-endpoint}?id={Id}`

The variant is the central entity. The **list** `product` endpoint returns
bare FK ids; this **single** endpoint returns nested objects plus embedded
child collections.

## Core fields

| Field | Type | Notes |
|---|---|---|
| `LanguageId` | string | locale |
| `Id` | string | variant id (= `ProductId` elsewhere) |
| `ProductIdView` | string | display/SKU id |
| `EAN` | string | barcode |
| `Name` | string | |
| `Limit` | int\|float | order limit |
| `Price` | float | |
| `Created` / `Changed` | string | timestamps |

## Nested relation objects

| Field | Shape |
|---|---|
| `Brand` | `{Id, …}` — brand id at `Brand.Id` |
| `Color` | object, nullable |
| `Model` | object (required) |
| `Shade` | object, nullable |
| `Size` | object, nullable |

On the list `product` endpoint these same relations are flat ids
(`Brand`, `Color`, `Model`, `Shade`, `Size`).

## Logistics fields (need composition)

| Source fields | Composed value |
|---|---|
| `Carton` | pieces per carton |
| `Depth`, `Width`, `Height`, `DimUM` | carton dimensions `D x W x H DimUM` |
| `Depth × Width × Height` | carton volume (m³) |
| `WeightBtto`, `Carton`, `WeightUM` | carton gross weight = `WeightBtto × Carton WeightUM` |
| `Weight`, `WeightUM` | unit net weight |
| `WeightBtto` | unit gross weight |
| `CustTariff` | customs tariff |
| `OriginName` | country of origin |
| `PackageInfo` | packaging text |

## Embedded child collections

Same data the bulk/pivot endpoints expose, scoped to this variant:

`Arrivals[]`, `Images[]`, `specifications[]`, `videos[]`, `statuses[]`,
`stickers[]`, `stocks[]`, `certifications[]`, `catalogPages[]`.

## Product images endpoint

Separate: `GET {base}/en/api/{product-images-endpoint}?id={Id}` →
array of images. English path only.
