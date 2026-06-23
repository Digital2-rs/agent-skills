# Promobox List Endpoints

Bulk endpoints. URL: `{base}/{lang}/api/{endpoint}` → JSON array of every
record. Same record set per language; only translatable fields differ by
`{lang}`. Keys are PascalCase. `Id` is the stable key. An empty-string FK
(`""`) means no relation (treat as null).

`*` marks a translatable field.

## Taxonomy / attributes

| Endpoint | Fields | Notes / links |
|---|---|---|
| `brand` | `Id` | `Id` is the brand name itself; not translated |
| `groups` (categories) | `Code`, `Name`*, `Parent`, `Sort` | `Parent` → category `Code` (self-referencing tree) |
| `color` | `Id`, `Name`*, `Image`, `HtmlColor` | `HtmlColor` = hex |
| `shade` | `Id`, `Name`*, `Image`, `HtmlColor` | `HtmlColor` = hex |
| `size` | `Id`, … | not translated |
| `status` | `Id`, `Name`*, `Image` | |
| `sticker` | `Id`, `Name`*, `Type`*, `Sort`, `Image` | `Type` also translatable |

## Model

| Endpoint | Fields | Links |
|---|---|---|
| `model` | `Id`, `Name`, `Description`*, `Image`, `ImageWebP`, `ImageHover`, `Sort`, `ExtDescr[]`, `GroupWeb1`, `GroupWeb2`, `GroupWeb3` | `GroupWeb1/2/3` → category `Code` (up to 3, many-to-many) |

- Identified by `Name` (the single-record endpoint takes `?id={Name}`),
  though each record also carries an `Id`.
- `ExtDescr[]` items are `{DescriptionType, Description}`;
  `DescriptionType === 'D'` is the video embed.
- Full rich single-record shape → `model.md`.

## Variant (product)

| Endpoint | Fields | Links |
|---|---|---|
| `product` | `Id`, `Name`, `Price`, `Created`, `Brand`, `Color`, `Size`, `Shade`, `Model` | each FK → that entity's `Id`; `Model` → model |

- FKs are bare ids on the list endpoint. The single-record endpoint
  returns nested objects → `variant.md`.

## Variant children & pivots

| Endpoint | Fields | Links |
|---|---|---|
| `productarrival` | `ProductId`, `Qty`, `Arrival` (date), `Value` (type) | `ProductId` → variant `Id` |
| `productstock` | `ProductId`, `Warehouse`, `Qty` | `ProductId` → variant `Id` |
| `productstatus` | `ProductId`, `StatusId` | pivot: variant ↔ status |
| `productsticker` | `ProductId`, `StickerId` | pivot: variant ↔ sticker |
