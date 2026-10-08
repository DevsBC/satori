# Decisions already closed

Read this when a tool looks reasonable and the layer already said no.
Each line is the choice, what was rejected, and why. Do not reopen these unless Satoru does.

The layer still wins on how to build. This file is the memory of what was tried and dropped.

## Backend

- Rust on Axum and Tokio. Nest, Express, and the rest of the README inventory are experience, not options for a new service.
- `infra/` is two crates: `persistence` (Firestore) and `secrets` (vault).
- OpenAPI is generated with utoipa from the handlers. A hand-written YAML drifts the day the route changes.
- Domain tests use an in-memory fake that implements the port. Libraries that record call counts (`mockall`) are out. A renamed helper must not fail a test.
- Live Firestore tests are `#[ignore]`. A shared dev project is not a fixture.
- The admin password is chosen at signup. A seed does not invent one.
- JWT TTL defaults to 604800 seconds. A missing vault value does not stop boot.
- `/health` is `{ "status": "ok" }`. A readiness probe that pings Firestore is per project, not part of the shell.

## Frontend

- Svelte apps are Svelte 5 with runes, created with Vite (`svelte-ts`). SvelteKit is out. `npm create svelte` is out.
- Navigation is `sv-router`, installed into that Vite app, routes declared in code. Its file-based routing is out, because it recreates `src/routes/` and fights features.
- Phone-only products use Ionic with Angular. Desktop, tablet, and mobile use Svelte. SaaS or admin uses Carbon, without Tailwind. A public app uses daisyUI, and daisyUI requires Tailwind.
- Firestore in the browser is initialized once from env. The web config is not a secret, but it is not hardcoded, so the next project does not inherit the previous `projectId`. A service-account JSON never ships in the frontend.
- Snapshots may use that client. Anything that needs the vault or a send credential goes through the Rust API.
- Context `"0"` on the dev server. Context `"1"` on the production build. No path hardcodes either one.

## What this file is not

It is not a scar log. The only production incident on record is in `README.md`: an agent deleted years of data while updating records. Other failures are not written here until Satoru states them. Do not invent them.
