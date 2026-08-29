---
name: sponsorunited-brand-deal-research
description: Research which properties a brand sponsors and what a league's deals look
  like, using the SponsorUnited API's search and deal-aggregation operations.
api: SponsorUnited API
operations:
- 82e1cade2feeb042d7132eb6df8da393
- b878b803278e85324c0e01ac4ccf1f66
- a195f821ab9ad56096e70493cf181f51
- 0345053225041f1a688be78ede2ee6ba
- 159c70a8643366b1c625b2794ed8a1e4
generated: '2026-08-29'
method: generated
source: openapi/sponsorunited-api-openapi.json
---

# Researching a brand's sponsorship footprint

Read-only. Every operationId below was verified present in the OpenAPI description at
`openapi/sponsorunited-api-openapi.json`.

## Before you start

- Base URL is `https://api.sponsorunited.com`; every path is rooted at `/api`.
- Send `Authorization: Bearer <jwt>`. Obtain it through the login flow, which is two-step
  and includes MFA — see `sponsorunited-authenticate.md`.
- There is no published rate limit and no `Retry-After` header. Pace yourself
  conservatively and back off on `429`, `502` and `503`.

## Steps

1. **Resolve the brand to an id.** `GET /api/search/{indexed_entity}`
   (operationId `82e1cade2feeb042d7132eb6df8da393`) runs a global search over a
   Meilisearch-indexed entity. Search the brand index by name.
   Identifiers are bare integers with no type prefix, so keep track of which entity type
   each id came from — an id alone will not tell you.

2. **Confirm the match.** `POST /api/search/{entity}/by-id/list`
   (operationId `b878b803278e85324c0e01ac4ccf1f66`) returns full records for a set of ids
   as a flat list. Use it to verify you picked the right brand before spending further
   calls.

3. **Pull the brand's deals for a season.** `POST /api/brand/{id}/properties/{year}`
   (operationId `a195f821ab9ad56096e70493cf181f51`) returns the properties a brand has
   deals with in that year. Its sibling
   `POST /api/brand/{id}/properties/metadata/{year}` (operationId
   `81ad3bfcad5cac9e389f292c5e33ef20`) returns aggregate counts and filter facets — call
   it first if you only need volume, since it is cheaper than the full list.

4. **Widen to the league.** `GET /api/league/deals`
   (operationId `0345053225041f1a688be78ede2ee6ba`) lists league deals per property with
   aggregated statistics, which is how you place one brand's footprint against its
   category. `GET /api/league/deals/filter-options` (operationId
   `159c70a8643366b1c625b2794ed8a1e4`) returns the full-dataset filter options — read it
   before constructing filters rather than guessing valid values.

## Paginating

Most list operations use `page` and `per_page` (default 10) and return the Laravel
envelope: `data`, `current_page`, `last_page`, `per_page`, `total`, `next_page_url`.
Stop when `next_page_url` is null — do not compute the last page yourself.
A few operations instead take `after_id`, an integer cursor meaning "records with id
greater than this". Check which parameters the operation actually declares.

## Handling failures

- `401 Unauthenticated` — by far the most common failure (222 operations declare it).
  Re-run the login flow and retry with a fresh token.
- `403` — permission-scoped and specific, e.g. "missing view news permission". Not
  retryable; the caller's role must change.
- `422 Validation error` — read the `errors` object for per-field detail.
- Errors do not use RFC 9457, and there are four different envelopes in this API
  (`{error}`, `{message}`, `{message, errors}`, `{error, message}`). Parse defensively;
  see `errors/sponsorunited-problem-types.yml`.
