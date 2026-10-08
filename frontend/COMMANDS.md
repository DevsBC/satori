# COMMANDS.md

Pass and fail are defined in `QUALITY.md`. This file is only the commands.

## Ionic + Angular

    npx ng lint
    npx ng test --watch=false
    npx ng build
    npx ng serve

## Svelte 5 on Vite

    npx svelte-check --tsconfig ./tsconfig.json
    npx vitest run
    npx vite build
    npx vite

## Locale keys

The project fails if the key sets differ. The check lives next to the catalogs and is wired into the pipeline above. Do not eyeball the JSON.
