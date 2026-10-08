# Frontend

Build rules for this layer. If this file and the root README disagree, this file wins.

## Which stack

First match wins. Do not mix them. Do not ask when the user message already matches a row.

| The product | Stack |
|-------------|--------|
| Mobile first, and the app is used entirely or almost entirely on a phone | Ionic with Angular |
| Desktop, tablet, and mobile, and the product is a SaaS or an admin tool | Svelte 5 on Vite, with Carbon (`carbon-components-svelte`) |
| Desktop, tablet, and mobile, and the product is for the general public | Svelte 5 on Vite, with daisyUI |
| None of the above is stated | Stop and ask once. A wrong kit is a rewrite. |

Svelte and Angular are the only frameworks. Ionic is the mobile shell on Angular, not a third framework.

A Svelte app is created with Vite, template `svelte-ts`. Dev and build go through Vite (`vite`, `vite build`). SvelteKit is not used. Do not add `@sveltejs/kit`, `+page.svelte`, or `+layout.svelte`. Do not scaffold with `npm create svelte`.

## Runes

Svelte 5 runes are the reactivity. `$state`, `$derived`, `$props`, `$effect`. Do not use `export let`, stores, or `$:` for component state. A store is legacy. New state is a rune.

Carbon and daisyUI are not combined. Carbon does not get Tailwind. daisyUI does not get Carbon components. A screen uses the kit's button, input, and modal before anyone draws a new one.

## Phase 1

The shell only. No product screen.

- The stack from the table.
- Light and dark themes on the same tokens.
- Locales `es` (default) and `en`, same keys, switch without a rebuild.
- URL navigation, session module, API client, and a Firestore instance initialized from env.
- Sign-in and sign-up views wired to the API. A 401 opens sign-in.
- The theme switch wired to the kit, as written under Theme.
- Quality pipeline once. Then phase 2.

## Navigation

Svelte navigation is [`sv-router`](https://github.com/colinlienard/sv-router). Install it in the Vite app. Do not scaffold with `npm create sv-router`. Do not turn on its file-based routing. Routes are declared in code, one per feature, so the folders stay feature-based. Why that library: the URL, the back button, and the route params stay typed, without SvelteKit.

Ionic uses the Angular router. Do not add a second one.

## Session and API

`shared/session` is the only app-wide state: theme, locale, and access token. Feature state stays in the feature. Components do not read `localStorage` themselves.

The token is persisted in `localStorage` under one key owned by that module. It is sent as `Authorization: Bearer`. It is never put in the URL and never logged. A 401 clears the session and navigates to `/signin`.

One HTTP client for the whole app. Svelte uses Axios. Angular uses `HttpClient`. Do not add Axios to an Ionic app, and do not call `fetch` from a feature. A component never imports the client. A feature calls a function in its own `api.ts`, and that function uses the shared client.

### Svelte: one Axios instance

Created once in `shared/api/client.ts`.

- `baseURL` is `VITE_API_BASE_URL`. No trailing slash. Paths are `/${context}/v1/...`.
- Request interceptor: if the session has a token, set `Authorization: Bearer <token>`.
- Response interceptor: the success value is `response.data`. A 401 clears the session. The error thrown to the feature is the backend `ApiError` (`code`, `message`, `details`), not the raw Axios error.
- Timeout: 30 seconds, same as the API. JSON only.

Phase 1 creates this instance and the interceptors. It does not add product endpoints. Phase 2 adds functions next to the feature:

```ts
export function getItem(id: string) {
  return api.get<Item>(`/${context}/v1/biblical-queries/${id}`);
}
```

The resource name comes from the task. The shape of the call does not.

### Angular: one HttpClient

An interceptor does the same two jobs: attach the bearer token, and on 401 clear the session. `environment.ts` holds the base URL. Feature services call `HttpClient`. Components call the service.

### Errors on screen

The UI shows the catalog key `error.<code>`. An unknown code shows `error.internal`. The server `message` is not rendered as the sentence the user reads. Why: the message is one language, and the catalogs are the translation.

## TypeScript

`strict` is on. No `any`. No `// @ts-ignore` and no `as unknown as` without a comment that says why, and without approval.

## Runes, continued

`$derived` computes values. `$effect` only talks to something outside the component (storage, a listener). Do not write state inside `$effect`. That pattern reruns forever and hides the data flow.

## A view has four states

Loading, empty, error, and the content. A blank screen is not a loading state. Empty and error use the catalog, not a hardcoded sentence.

## Components

- Feature-based. A feature owns its route and the components only that feature uses.
- A component moves to `shared` on the second feature that needs it. Not before. A shared folder of unused widgets is a junk drawer.
- The component's own view, logic, and style files sum to at most 300 lines. Spec files and locale JSON do not count. Split by responsibility when it grows. Do not split to game the number.
- Logic that is not about rendering lives outside the component. The component receives data and emits events.
- No user-visible string literal in a component. It goes through i18n.
- No raw color in a component. It uses a token.

## Theme

One switch controls the theme. It persists on the device. The default follows the system until the user chooses.

Both themes define the same tokens. A missing token in one theme is a bug.

    --color-bg
    --color-surface
    --color-text
    --color-text-muted
    --color-border
    --color-primary
    --color-danger

Text and color stay on those tokens so a label and its surface change together. A component does not mention `gray-700` or a hex.

The switch sets `document.documentElement.dataset.theme` to `light` or `dark` and writes the matching tokens. One mechanism. The kit does not get a second toggle.

daisyUI requires Tailwind. Carbon does not. Do not add Tailwind to a Carbon app.

daisyUI: the theme file defines daisy themes `light` and `dark` so `base-100` is `--color-bg`, `base-content` is `--color-text`, and `primary` is `--color-primary`. Screens use daisy component classes (`btn`, `input`). Those classes follow `data-theme`. A screen does not also set a hex.

Carbon: light is the Carbon theme `white`. Dark is `g100`. The app root renders `<Theme theme="white">` or `<Theme theme="g100">` from the same switch. Screens use Carbon components. The tokens above are set in the same switch so any custom CSS matches the Carbon surface.

## Auth views

Phase 1 includes `features/auth`. It is shell, not a product feature.

- `/signin` posts `{ email, password }` to `/${context}/v1/auth/signin`. On 200 it stores `token` and navigates to `/`.
- `/signup` posts the same body to `/${context}/v1/auth/signup`. On 201 it navigates to `/signin`. It does not store a token. The API does not return one.
- Fields are the kit's inputs. Labels and errors come from the catalogs. Unknown JSON fields are not sent.
- `/` is the sample view until phase 2 replaces it.

## i18n

- Catalogs: `es.json` and `en.json`. Default locale `es`. Fallback `en`.
- The key set is identical. A key in only one file fails the pipeline.
- The user can switch language at runtime. Compile-time-only i18n is not enough, because a language change would require a rebuild.
- Angular: `ngx-translate`. Svelte: `svelte-i18n`.

Why those two: both load the catalogs at runtime. The official Angular localizer does not.

## Firebase in the shell

Phase 1 initializes Firestore. The app is ready for a snapshot without a second round of setup. Product screens still go through `shared/api` unless the task says that screen listens to Firestore.

The browser SDK lives in `shared/firebase`. Components do not import Firebase. There is one app and one Firestore instance.

The web config is not a secret. Hardcoding it inside a feature is still wrong: the next project generated from this shell would ship the previous `projectId`. One module reads the values. Everything else imports the ready Firestore.

Vite, in `.env`, never committed. `.env.example` lists the keys empty:

    VITE_FIREBASE_API_KEY
    VITE_FIREBASE_AUTH_DOMAIN
    VITE_FIREBASE_PROJECT_ID
    VITE_FIREBASE_APP_ID
    VITE_FIREBASE_MESSAGING_SENDER_ID

Angular reads the same keys from `environment.ts`, which is generated from those env vars at build time. It is not a second handwritten config.

A service-account JSON never enters this repo. That key is the admin SDK. In Vite it would be inside the bundle.

## Context

One function, `context()`, used by the API client and by Firestore. No path hardcodes `"0"` or `"1"`.

- Dev server (`vite`, `ng serve`): `"0"`.
- Production build (`vite build`, `ng build` for production): `"1"`.

Why: the local API rejects `"1"`. The deployed API is where `"1"` belongs. A production bundle pointed at a local API will receive 400, and that is correct.

**Snapshots.** A feature asks `shared/firebase` for a listener. It does not call `onSnapshot` itself. The path is `{APP_NAME}/{context}/{collection}/{id}` with that same `context()`. Collection names stay plural, lowercase, and hyphenated. The task names which collections are listened to. The others stay on the API.

**What stays on the API.** Anything that needs the vault, a role check, or a secret. Sending a push notification is that case: the client may read a device token and `POST` it to the Rust API. The client does not hold the send credential.

Why the split: a snapshot is a read the screen can hold open. A secret is not. The API already has the vault for the second one.

## Forms and dates

Inputs are the kit's inputs. Each control has a label. An icon-only button has a translated `aria-label`. The focus ring uses a token.

Dates arrive as ISO 8601 and are shown with `Intl.DateTimeFormat` for the active locale. Do not add a date library for that.

## Why, not what

Same rule as the backend. A comment says why. `missing_docs` is not a gate. The kit's own documentation is not copied into the repo.
