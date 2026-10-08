# Data model

## Mandatory base for every document

| Field        | JSON / Firestore | Rust field     | Rule                          |
|--------------|------------------|----------------|-------------------------------|
| id           | id               | id             | UUID v4 string                |
| creationDate | creationDate     | creation_date  | ISO 8601 UTC, set on create   |
| updateDate   | updateDate       | update_date    | ISO 8601 UTC, set on every write |
| context      | context          | context        | `"0"` or `"1"`                |

Serde maps Rust `snake_case` to JSON `camelCase`. Do not hand-rename these four.

## Firestore path

    {APP_NAME}/{CONTEXT}/{COLLECTION}/{DOCUMENT_ID}

Example:

    my-app/0/biblical-queries/02aa9331-c94b-4182-8e7e-68664827a12f

## Rules

- Collection name is plural, lowercase, and hyphenated. Words separate with `-`, as in `biblical-queries`. Not `camelCase`, not `snake_case`.
- A single word stays a single word (`users`). The product chooses the name, in that shape.
- ID always UUID v4.
- Dates are Chrono UTC values. JSON and Firestore store ISO 8601.
- Document `context` equals path context.
- Never write without the 4 base fields.
- One helper in `infra/persistence` builds paths. Hand-built paths are forbidden.

    fn doc_path(app: &str, ctx: &str, col: &str, id: &str) -> String

## Context

- Deployed (Cloud Run): paths may use `"0"` or `"1"`, matching the request.
- Local process, local seeds, and tests: only `"0"`.
- `"1"` on a local process is rejected before any write (HTTP 400). See `ENDPOINTS.md`.
- No Firestore emulator. Local dev talks to the dev GCP project, context `"0"` only.

## Tenant

- No dynamic multi-tenant.
- `APP_NAME` is a constant compiled into the service.
- Each tenant is its own service.
