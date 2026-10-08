# Endpoints

## Format

    {host}/{context}/{version}/resource/action

- `{context}`: `"0"` or `"1"`.
- `{version}`: `v1`, `v2`, … If the version segment is absent, it is `v1`.
- Exception: health checks. No context, no version.

## Examples

Required on every API:

- POST /0/v1/auth/signin
- POST /0/v1/auth/signup
- POST /0/v1/users/{id}/role
- GET  /health
- GET  /docs
- GET  /api-docs/openapi.json

A product route uses the same shape and a hyphenated resource, for example `GET /0/v1/biblical-queries/{id}`. The resource name comes from `CURRENT_TASK.md`.

## Who must be authenticated

Public, no JWT: `GET /health`, `GET /docs`, `GET /api-docs/openapi.json`, `POST /{context}/{version}/auth/signup`, `POST /{context}/{version}/auth/signin`.
Every other route requires `Authorization: Bearer`. Missing or bad token: `unauthenticated` (401).
A body that is not JSON, or JSON that does not match the DTO: `invalid` (400), same `ApiError` shape. Not 500.

## Local and deployed

The default build is local. It rejects context `"1"`.
`docker/runtime/api.Dockerfile` is the only build that passes `--features deployed`. That binary accepts `"0"` and `"1"`.
There is no env var to switch this. `cargo run`, `dev.ps1`, and `docker/dev` do not enable `deployed`.

## Where the binary runs

- **Deployed:** accepts `"0"` and `"1"`. The deployment's GCP project is the data boundary. Do not point a prod deployment at the dev project or the reverse.
- **Local** (default build): middleware rejects context `"1"` with `invalid` (400) before a handler or a repository runs.

## Health

- `GET /health` → 200 and `{ "status": "ok" }` when the process is up. No context, no version, no JWT, no dependency check.
- A readiness route that pings Firestore or the vault is not part of this standard. Add it in the project that needs it.

## OpenAPI

The spec is generated from the handlers. There is no hand-written `openapi.yaml` and no second binary. A YAML checked into git drifts from the route the day someone edits only one of them.

Crates, in `api` only: `utoipa`, `utoipa-axum`, `utoipa-swagger-ui`. Domain entities are not schemas. DTOs are, with `ToSchema`.

- Each handler has `#[utoipa::path]`: method, path, tag, request body, success body, and every `ApiError` status that handler can return.
- `operation_id` is the handler name. The tag is the resource (`auth`, `users`, `health`).
- The router is an `OpenApiRouter` merged with the Axum router, so a route cannot exist without a path annotation.
- Swagger UI: `GET /docs`. Spec: `GET /api-docs/openapi.json`. No context segment, no version segment, no JWT. Same reason as `/health`: the contract must be readable before a token exists.
- A test in `api` builds the `OpenApi` value and asserts it serializes to JSON. That test is part of `cargo test`. It does not bind a port.

Why the annotation lives on the handler: the HTTP contract and the function that implements it change in the same diff.

## Listen port

- Env `PORT`, default `8080`.
- Local, Docker, and Cloud Run use that same port.
- Do not bind `3000`.

## Context middleware

- Extract `:context`.
- Not `"0"` or `"1"` → 400 `invalid`.
- Local and the value is `"1"` → 400 `invalid`.
- Otherwise store it on the request state. Repositories take it from there, not from a global.
