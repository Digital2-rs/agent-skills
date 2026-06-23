# Promobox Auth

OAuth2 password grant.

## Token request

`POST {base}/token`, form-encoded:

| Field | Value |
|---|---|
| `grant_type` | `password` |
| `username` | service account |
| `password` | service account |

Response:

```json
{ "access_token": "…", "expires_in": 3600 }
```

`expires_in` is seconds.

## Using the token

Every data call sends `Authorization: Bearer {access_token}`.

## Caching

Cache the token until it expires; reuse across requests. On a `401`,
fetch a fresh token once and retry. Do not request a new token per call —
the token endpoint is rate-relevant and the token is valid for the full
`expires_in` window.

## Config

`base`, `username`, `password` come from `services.promobox.*`
(`PROMOBOX_BASE_URL`, `PROMOBOX_USERNAME`, `PROMOBOX_PASSWORD`).
