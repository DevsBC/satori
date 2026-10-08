# Folder structure

The 300-line rule is in `FRONTEND.md`. This file only places files. Feature names come from the product, not from this tree.

## Shared shape

    src/
      features/
        <feature>/
          <feature>.routes.*
          components/
      shared/
        ui/
        theme/
        i18n/
          es.json
          en.json
        session/
        api/
        firebase/
      app/

`shared/ui` is empty in phase 1 unless the sample button is wrapped. Do not invent `Button`, `Card`, and `Modal` when the kit already has them.

`app/` is the shell: bootstrap, theme switch, locale switch, and the active feature.

## Ionic + Angular

    src/app/
    src/features/auth/
    src/shared/ui/
    src/shared/theme/
    src/shared/i18n/
    src/shared/session/
    src/shared/api/
    src/shared/firebase/
    angular.json
    ionic.config.json
    capacitor.config.ts    # only if the task ships to a device store

Capacitor is not part of phase 1. Add it when the task says the app is installed on a device.

## Svelte 5

    src/main.ts
    src/app/
    src/features/auth/
    src/shared/ui/
    src/shared/theme/
    src/shared/i18n/
    src/shared/session/
    src/shared/api/
    src/shared/firebase/
    vite.config.ts
    svelte.config.js

`src/app/` mounts `sv-router`. There is no `src/routes/` and no Kit file. The screen lives in the feature. The route table points at those features.

## What does not go here

A feature does not import another feature. It imports `shared` and the kit. If two features need the same data shape, that shape is promoted to `shared` for a reason you can say in one sentence.
