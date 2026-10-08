# QUALITY.md — Frontend

A task is not done until this pipeline passes. No warnings left behind.

## Pipeline

Ionic + Angular, from the project root:

    npx ng lint
    npx ng test --watch=false
    npx ng build

Svelte 5 on Vite, from the project root:

    npx svelte-check --tsconfig ./tsconfig.json
    npx vitest run
    npx vite build

Then, for either stack, the locale check and the line check below.

If a step fails, fix it. Do not skip ahead. After a fix, re-run from the first step. Paste the raw output.

## Locale keys

`es.json` and `en.json` have the same key set. A script or a test fails when they differ. Why: a missing English key shows up in production as a raw id, and a missing Spanish key does the same for the default locale.

## 300-line check

Sum the lines of each component's view, logic, and style. Spec files and JSON catalogs do not count. Fail when the sum is over 300.

## Tests

Same rule as the backend. One behavior, one test, named for the behavior. Do not assert private functions. Do not snapshot a whole page to freeze markup. A snapshot fails when a token or a translated word changes, and then nobody trusts the suite.

Phase 1 tests the shell only: both themes define the same tokens, and the two catalogs have the same keys. Phase 2 adds a test when a feature rule can break without a compile error. One rule, one test.

## Gates

- No hex color and no palette utility that bypasses the tokens, except inside the theme file that defines the tokens.
- No user-visible string outside the catalogs.
- `strict` TypeScript. No `any`.
- Svelte routing is `sv-router` only, declared in code, not file-based. No second router on Angular.
- No HTTP call from a component. Svelte features go through the shared Axios instance. Angular features go through `HttpClient`. No direct `localStorage` outside `shared/session`.
- Firestore is initialized once in `shared/firebase`. No service-account file. Web config comes from env, not from a literal in a feature.
- No second UI kit.
- No new dependency without approval. The kits named in `FRONTEND.md` are already approved.
