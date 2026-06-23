---
name: promobox-api-reference
description: >
  Reference for the Promobox product catalog API — what each endpoint
  returns and how the entities relate. Use when fetching or mapping
  Promobox data: brands, categories (groups), colors, shades, sizes,
  statuses, stickers, models, variants (products), arrivals, stocks.
  Triggers on the OAuth token flow, the per-language /{lang}/api/{endpoint}
  URL shape, the ?id= single-record endpoints, PascalCase JSON keys (Id,
  Name, HtmlColor, GroupWeb1, ProductId, ExtDescr), or questions about
  which field links to which.
---

# Promobox API

External product catalog. Read-only — the API is the source of truth.

## How it works, in one breath

Two URL shapes: **list** `{base}/{lang}/api/{endpoint}` returns every
record (text fields translated per `{lang}`); **single**
`{base}/{lang}/api/{endpoint}?id={x}` returns one rich record with nested
relations. JSON keys are PascalCase; `Id` is the link key.

## Quick Reference

### 1. Auth → `rules/auth.md`

- `POST {base}/token`, form `grant_type=password` + `username` + `password`
- Response `{access_token, expires_in}`; `Authorization: Bearer {token}` on every call
- Token cached until expiry; refetch on 401

### 2. List endpoints (bulk) → `rules/list-endpoints.md`

- `brand` · `groups` (categories) · `color` · `shade` · `size` · `status` · `sticker` · `model` · `product` (variant)
- Pivots/children: `productarrival` · `productstock` · `productstatus` · `productsticker`
- Each returns a JSON array of every record; `Name`/`Type`/`Description` are translatable per `{lang}`
- FK fields hold the related entity's `Id`; empty string `""` = no relation

### 3. Model (single, rich) → `rules/model.md`

- `?id={Name}` — models are keyed by `Name`
- Rich fields: `ExtDescr[]` (`DescriptionType 'D'` = video), `Colors[]`, `ALTModel[]` (alternatives), `KOMPModel[]` (complementary), `AdditionalInfo[]`, `TechDim`
- Grouped into up to 3 categories via `GroupWeb1/2/3` → category `Code`

### 4. Variant / product (single, rich) → `rules/variant.md`

- List `product`: `Id, Name, Price, Created, Brand, Color, Size, Shade, Model` (bare ids)
- Single `?id={Id}`: nested `Brand/Color/Model/Shade/Size` objects + embedded `Arrivals/Images/statuses/stickers/stocks/specifications`
- Carton/weight fields need composition (`Depth×Width×Height`, `WeightBtto×Carton`)
- Hub entity: 1 model + brand/color/shade/size; many arrivals/stocks/statuses/stickers

### 5. Relationships → `rules/relationships.md`

- `variant` is the hub; `model` is the product family grouped into categories
- `category.Parent` → `category.Code` (self tree)
- `arrival`/`stock` → variant via `ProductId`; `productstatus`/`productsticker` are pivots
- Full link-key table + diagram

## How to apply

1. Need a field name or link key fast → read the Quick Reference above.
2. Need full DTO shape or field-composition detail → open the matching `rules/*.md`.
3. Field names and keys are exact (PascalCase) — match them verbatim when mapping.
