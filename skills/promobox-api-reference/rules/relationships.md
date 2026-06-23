# Promobox Relationships

## Link-key table

| From | Field | To | Cardinality |
|---|---|---|---|
| category | `Parent` | category `Code` | self-tree (one parent) |
| model | `GroupWeb1/2/3` | category `Code` | many-to-many (≤3) |
| variant (product) | `Model` | model | many variants → one model |
| variant | `Brand` | brand `Id` | many → one |
| variant | `Color` | color `Id` | many → one (nullable) |
| variant | `Shade` | shade `Id` | many → one (nullable) |
| variant | `Size` | size `Id` | many → one (nullable) |
| arrival | `ProductId` | variant `Id` | many → one |
| stock | `ProductId` | variant `Id` | many → one |
| productstatus | `ProductId` + `StatusId` | variant ↔ status | pivot (m2m) |
| productsticker | `ProductId` + `StickerId` | variant ↔ sticker | pivot (m2m) |

Empty-string FK (`""`) = no relation.

## Map

```
category ──Parent──▶ category          (self-referencing tree)
   ▲
   │ GroupWeb1/2/3  (many-to-many: model → category, up to 3)
model ◀──────────────┐
   ▲ Model           │
   │                 │
variant (product) ───┼─ Brand ──▶ brand
   │                 ├─ Color ──▶ color   (nullable)
   │                 ├─ Shade ──▶ shade   (nullable)
   │                 └─ Size  ──▶ size    (nullable)
   ├─ ProductId ◀──── arrival   (many)
   ├─ ProductId ◀──── stock     (many)
   ├─ productstatus  ──▶ status   (pivot, many-to-many)
   └─ productsticker ──▶ sticker  (pivot, many-to-many)
```

## Reading the graph

- **variant** is the hub: belongs to one model + one each of
  brand/color/shade/size; has many arrivals, stocks, statuses, stickers.
- **model** is the product family — the thing shown on a product page;
  variants are its buyable SKUs. Grouped into up to 3 categories.
- **category** is a tree via `Parent` → `Code`.
- `ProductId` everywhere = the variant `Id`.
