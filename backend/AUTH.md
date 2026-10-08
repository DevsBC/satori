# Auth

## Password hashing

- Algorithm: Argon2id, crate `argon2`, default parameters. Do not tune them without approval.
- Salt: random per password.
- Hash stored on the user document as `passwordHash`. Never log the password or the hash.
- Accounts live in the Firestore collection `users`.
- The person chooses the password at signup. Nothing in the vault or the seed predefines it.

## Password and email

- Length: 6 to 128 characters. No composition rules.
- Email is trimmed, then stored lowercase. It must contain one `@`, with a non-empty local part and a non-empty domain. No other format check.
- The server generates `id` (UUID v4). A client `id` in the body is rejected with `invalid` (400).
- Auth DTOs deny unknown JSON fields.
- Email is unique inside one context. A second signup with the same email is `conflict` (409).
- These limits are named constants in `domain`, not literals in the handler.

## Roles

The JSON field is `role`. The app may store other strings. Three meanings are reserved:

| Stored value | Meaning |
|--------------|---------|
| missing, or `user` | Basic user |
| `admin` | Delegated operator |
| `superadmin` | The account registered with `SATORU_ADMIN_EMAIL` |

An unknown string is authorized as a basic user. It is never treated as `admin` or `superadmin`.

- Public signup body is `email` and `password` only. A `role` in that body is ignored.
- If the email equals vault `SATORU_ADMIN_EMAIL`, the stored role is `superadmin`. Otherwise it is `user`.
- Only an authenticated `superadmin` may grant or revoke `admin`, via `POST /{context}/{version}/users/{id}/role` with body `{ "role": "admin" }` or `{ "role": "user" }`.
- That route rejects `superadmin` in the body with `invalid` (400). No endpoint assigns `superadmin`. That role exists only through the vault email match on signup.
- Unknown user id on that route: `not_found` (404). Caller is not `superadmin`: `forbidden` (403).
- Signup does not change the role of an email that already exists.

## JWT

- Algorithm: HS256. Crate: `jsonwebtoken`.
- Secret: vault document `JWT_SECRET`.
- TTL: 604800 seconds. Vault `JWT_TTL_SECS` overrides it only when the value is an integer greater than 0. Missing or invalid keeps 604800. The process does not fail boot over TTL.
- Claims: `sub` (user id), `ctx` (`"0"` or `"1"`), `role` (stored role, or `user` when missing), `iat`, `exp`.
- Header: `Authorization: Bearer {token}`.
- Expired, missing, or bad signature: `unauthenticated` (401).
- Authenticated but not allowed for the action: `forbidden` (403).

## Bodies

Signup `POST .../auth/signup`. Unknown fields rejected.

    { "email": "person@example.com", "password": "secret" }

Status 201. No token. The hash is not in the response.

    { "id": "02aa9331-c94b-4182-8e7e-68664827a12f", "email": "person@example.com", "role": "user" }

Signin `POST .../auth/signin`. Same request fields.

Status 200. `expiresIn` is the TTL in seconds that was signed into the token.

    { "token": "<jwt>", "tokenType": "Bearer", "expiresIn": 604800 }

Role `POST .../users/{id}/role`. Caller is `superadmin`.

    { "role": "admin" }

`"user"` revokes admin. `"superadmin"` is `invalid` (400). Status 200:

    { "id": "02aa9331-c94b-4182-8e7e-68664827a12f", "role": "admin" }

Unknown email or bad password on signin: `unauthenticated` (401), same message for both.
