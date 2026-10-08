# Backend

Build rules for this layer. If this file and the root README disagree, this file wins.

## Glossary

- **context** — `"0"` (dev) or `"1"` (prod). Same value in the URL, the Firestore path, and the document.
- **APP_NAME** — tenant id. A string constant compiled into the service, not an env var. One deployed service per tenant. No dynamic multi-tenant.
- **vault** — Firestore documents that hold secrets. See `VAULT.md`.
- **seed** — idempotent create-if-missing for base data. See `SEED.md`.
- **port** — a trait in `domain` that `infra` implements. Domain never imports infra.
- **handler** — HTTP function in `api`. No business rule.
- **use case** — application function in `domain`. Orchestrates entities and ports.

## Principles

1. Clean Architecture in Rust. HTTP: Axum. Runtime: Tokio.
2. Crates: `api`, `domain`, and one crate per folder under `infra/`. See `FOLDER_STRUCTURE.md`.
3. No Rust file exceeds 150 lines. The check and the count rule live in `QUALITY.md`.
4. GCP by default: Firestore + Cloud Run.
5. Secrets in the vault. Bootstrap env is only GCP credentials and `GCP_PROJECT_ID`.
6. Deployed, the binary serves `"0"` and `"1"`. Local process rejects `"1"` with 400. See `ENDPOINTS.md`.
7. Docker for dev and runtime. See `DOCKER.md`.
8. Idempotent seeds. See `SEED.md`.
9. The vault document `SATORU_ADMIN_EMAIL` decides who becomes `superadmin` on signup. See `AUTH.md`.
10. Auth: JWT HS256 + Argon2id. See `AUTH.md`.
11. Phase 1 is the boilerplate below. Phase 2 is the product. See `AGENTS.md`. Do not ask during phase 1.

## Dependency rule

    api → domain ← infra/*

- domain: entities, use cases, ports, errors. No Firestore, no HTTP, no JWT, no env vars.
- each crate under `infra/`: implements ports. Depends on `domain`, never on `api`.
- api: HTTP, DTOs, handlers, middleware. Depends on `domain` and only the infra crates it calls.

## Request path

A handler reads the DTO, calls one use case, and maps `DomainError` to `ApiError`. It does not apply a business rule and it does not touch Firestore.
The use case calls ports. It does not know HTTP.
A repository implements a port. It does not choose a status code.

The HTTP body is a DTO in `api`. The entity in `domain` is not the HTTP contract. The handler maps one to the other. Serde `camelCase` applies to the DTO and to the Firestore document.

## What goes in each crate

| Crate | Contains | Does not contain |
|-------|----------|------------------|
| api | routes, handlers, DTOs, state, main, middleware | business rules |
| domain | entities, use cases, ports, errors | Firestore, HTTP, JWT, env |
| infra/persistence | Firestore repos, paths, seeds | HTTP, DTOs |
| infra/secrets | vault reader and cache | HTTP, DTOs |

## Names

- Rust identifiers: `snake_case`.
- JSON on the wire and in Firestore: `camelCase` via serde.
- Base fields serialize as `id`, `creationDate`, `updateDate`, `context`.

## Business rules

A threshold, role name, or limit is a named `const` or a type in `domain`.
`api` and `infra` do not compare against a business literal.
The use case references the name. When the rule changes, the diff is that name.

## Errors

- domain defines `DomainError` with `thiserror`.
- infra wraps driver failures in `InfraError` and maps them to `DomainError`. Infra does not choose HTTP status.
- api maps `DomainError` to one `ApiError` body: `code`, `message`, `details`.
- `details` is omitted when empty. `code` is snake_case.

| code | HTTP |
|------|------|
| invalid | 400 |
| unauthenticated | 401 |
| forbidden | 403 |
| not_found | 404 |
| conflict | 409 |
| internal | 500 |

No other codes unless the operator adds a row here first.

## Outcomes, not versions

Versions live in the project's `Cargo.lock`, which is committed. This file does not pin them.
A new crate or a channel change needs approval. Channel is stable, named in `rust-toolchain.toml`.
Workspace resolver is `"2"`.

- HTTP: Axum. Async runtime: Tokio. Trace and a 30-second request timeout: `tower-http`.
- CORS is not permissive. Phase 1 ships without a CORS layer. Phase 2 adds origins only when `CURRENT_TASK.md` lists them. Do not ask, and do not use a permissive layer.
- Async ports in domain: `async-trait`.
- JSON: Serde. Timestamps: Chrono, UTC, serialized as ISO 8601. Ids: `uuid` v4.
- Domain errors: `thiserror`. Logs: `tracing`. The subscriber uses `env-filter`, `json`, and `fmt`, not the `time` feature.
- Bootstrap env: `dotenvy` loads `.env`, `envy` maps it. Allowed fields: `GOOGLE_APPLICATION_CREDENTIALS` and `GCP_PROJECT_ID`.
- Firestore client: crate `firestore`, only inside `infra/`.
- Passwords: Argon2id via `argon2`. JWT: HS256 via `jsonwebtoken`.
- OpenAPI: `utoipa` in `api` only. See `ENDPOINTS.md`.

`domain` may depend on `serde`, `chrono`, `uuid`, `thiserror`, and `async-trait`.
It does not depend on `firestore`, `axum`, `jsonwebtoken`, `utoipa`, `dotenvy`, or `envy`.

## Why, not what

`cargo doc --workspace --no-deps` is the autodoc. No Compodoc and no second generator. Rustdoc already builds HTML from the crate; another tool would document a different story than the one the compiler checks.

A `///` comment exists only to answer why: the constraint, the rejected alternative, or what breaks if this changes. What the function does is the code. Do not narrate the signature (`/// Creates a user`). Do not require a comment on every item. A comment that restates the name is noise, and `missing_docs` is not a CI gate.

Module docs (`//!`) belong on `domain` and on each port. They say why the boundary is there, not a tour of the files.

## Boilerplate (phase 1)

Finish this set before any product file. It is functional when `cargo test` is green offline. No server tour, no live Firestore call.

- Workspace: `api`, `domain`, `infra/persistence`, `infra/secrets`. Resolver `"2"`. `Cargo.lock` committed.
- `APP_NAME` constant. Path helper. Vault reader with cache. Context middleware. `deployed` feature, off by default.
- Routes: `/health` returning `{ "status": "ok" }`, signup, signin, `users/{id}/role`, `/docs`, `/api-docs/openapi.json`. Nothing else. No readiness probe.
- Every one of those handlers annotated in the OpenAPI spec. The serialize test passes.
- Collection `users` with the base fields, `email`, `passwordHash`, `role`.
- Error map, DTO versus entity, 30-second timeout, tracing, `.env.example`, `dev.ps1`, both Dockerfiles, compose.
- Quality pipeline once. Then phase 2, or stop if there is no product objective.

## Testing

Rules, fakes, and the closed list of phase 1 behaviors are in `QUALITY.md`.
