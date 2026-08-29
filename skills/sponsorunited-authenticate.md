---
name: sponsorunited-authenticate
description: Obtain and use a SponsorUnited API bearer token through the two-step login
  and MFA flow.
api: SponsorUnited API
operations:
- 535ffeda4b78916efd658b2844ea897a
- 5c72ebefedf27de4e2552c970a09eeb9
- 5f4adbec6a6eedd8be6332f0b05cabb8
- 8e5dc3654b57a13f0272f04962926281
generated: '2026-08-29'
method: generated
source: openapi/sponsorunited-api-openapi.json
---

# Authenticating against the SponsorUnited API

The API declares two security schemes. Which one you use is not a preference — they are
for different callers.

- **`bearerAuth`** — HTTP bearer, `bearerFormat: JWT`. The scheme for platform users.
  This is the one you want.
- **`apiKeyAuth`** — an `X-API-Key` header the spec describes as a "Service API key for
  external services (ai-api, chat-api)", generated with `php artisan su:api-token:generate`.
  That is an internal server-side command, so this scheme is not something an external
  consumer can obtain. Do not build against it.

There is no OAuth2 flow and therefore no scopes. Authorization is role- and
ownership-based, surfaced as `403` at call time rather than as a grantable scope.

## Steps

1. **Log in.** `POST /api/auth/login` (operationId `535ffeda4b78916efd658b2844ea897a`).

2. **Request an MFA PIN.** `GET /api/auth/pin/send/{mfaOption}`
   (operationId `5c72ebefedf27de4e2552c970a09eeb9`). This operation declares `429`
   "Maximum attempts reached" — it is attempt-capped, so do not loop on it.

3. **Verify the PIN.** `POST /api/auth/pin/verify`
   (operationId `5f4adbec6a6eedd8be6332f0b05cabb8`). Also `429`-capped, with
   "Invalid or expired PIN / Maximum attempts reached".

4. **Send the token.** Put it in `Authorization: Bearer <jwt>` on every subsequent call.

5. **Log out when done.** `GET /api/auth/logout`
   (operationId `8e5dc3654b57a13f0272f04962926281`).

## Related operations

- `GET /api/email-exists` (operationId `345afa474b4097d1f2b85998d47f77c8`)
- `POST /api/auth/passwords/email` (operationId `d35a865c89c7b6977ccbfffbf4dbf5a4`) —
  send a password reset email
- `POST /api/auth/passwords/reset` (operationId `af7b3ac0f01be02abbf5b940f6c10e96`)

## Cautions

- No token lifetime is published. Treat `401` as your expiry signal and re-authenticate.
- Neither `429` response declares a `Retry-After` header, so you have no published
  backoff interval. Wait generously before restarting the login flow.
- Getting an account at all requires a paid subscription arranged through sales — there is
  no self-serve sign-up. See `plans/sponsorunited-plans-pricing.yml`.
