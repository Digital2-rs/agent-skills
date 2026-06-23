# Promobox Model (single record)

`GET {base}/{lang}/api/{model-endpoint}?id={Name}`

Models are keyed by **`Name`** — pass the name as `id`. Returns one rich
record.

## Fields

| Field | Type | Notes |
|---|---|---|
| `LanguageId` | string | locale of this response |
| `Id` | string | numeric model id |
| `Name` | string | the key |
| `Description` | string | translatable |
| `Description2` | string | secondary description |
| `Image` | string | |
| `ImageWebP` | string | |
| `ImageGif` | string | |
| `ImageHover` | string | |
| `Video` | string | |
| `Sort` | int | |
| `ExtDescr[]` | array | extended descriptions, see below |
| `Colors[]` | array | the model's color variants |
| `ALTModel[]` | array | alternative models |
| `KOMPModel[]` | array | complementary models |
| `PrintInfo[]` | array | print/branding info |
| `AdditionalInfo[]` | array | extra info + images, see below |
| `ImageTeh` / `ImageTehWebP` | string | technical drawing image |
| `TechDim` | string | derived (see below) |

## ExtDescr[]

Items: `{DescriptionType, Description}`.

- `DescriptionType === 'D'` → the **video embed** HTML/URL.
- Other types carry additional descriptive blocks.

## AdditionalInfo[] and TechDim

`AdditionalInfo[]` items carry an `Image`. `TechDim` is the technical
dimensions drawing: the first `AdditionalInfo` image whose name contains
`_dim` or `_101`.

## Relations

- `GroupWeb1/2/3` (from the **list** `model` endpoint) → category `Code`,
  up to 3, many-to-many. The single-record response focuses on media and
  related-model arrays rather than category codes.
- `Colors[]` / `ALTModel[]` / `KOMPModel[]` reference other models/colors
  for cross-sell and selection UI.
