# Seed

## Goal

Base data in Firestore, created once per context, never duplicated.

## Mechanism

- One runner per context.
- Each seed checks existence.
- If missing, it creates the document.
- If present, it does nothing. It does not update and it does not delete.
- Runs on service start or via an explicit command.

## Where it lives

Seeds live in `infra/persistence`. One runner, one file per seed.

## What a seed is not

A seed does not create users and does not set passwords.
The first `superadmin` is a signup. See `AUTH.md`.

Each seed implements:

    trait Seed {
        fn name(&self) -> &str;
        async fn run(&self, ctx: &str) -> Result<()>;
    }

## Idempotency

Prefer a deterministic id. If the id cannot be deterministic, write a marker at `seed/{name}` in that context and skip when the marker exists.

## Local

A seed started from a laptop or from dev compose receives context `"0"` only.
Seeding `"1"` requires an explicit operator approval and a deployed run, not a local one.

## Rules

- A seed never deletes.
- A seed never modifies existing documents.
- A seed is safe to run on every deploy of that context.
