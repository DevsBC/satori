# Folder structure

Where crates live. Not a catalog of the product's files.
Names inside `handlers/`, `entities/`, and `use_cases/` come from the product in `CURRENT_TASK.md`.
Health and the routes in `AUTH.md` are minimum boilerplate. Other handlers exist only when `CURRENT_TASK.md` names them.

The 150-line rule is in `QUALITY.md`.

## Workspace

    api/
      src/
      Cargo.toml
    domain/
      src/
      Cargo.toml
    infra/
      persistence/          # Firestore
      secrets/              # vault
    docs/                   # human docs; not the agent contract
    Cargo.toml
    Cargo.lock              # committed; this is the version pin
    dev.ps1
    docker/
      dev/api.Dockerfile
      runtime/api.Dockerfile
      docker-compose.yml
    .env.example
    .gitlab-ci.yml          # runs the QUALITY.md pipeline

`target/` is build output. It is not committed.
Do not add files under `docs/` to finish a code task.
`.agents/` and `.cursor/` are tool folders, not application code.

## Dependency

Each folder under `infra/` is a crate. It depends on `domain` and not on `api`.
`api` depends on `domain` and only on the infra crates it calls.
Those two crates are the minimum boilerplate: Firestore and the vault.
Another infra crate is a decision for that product. It is not part of this standard, and it is not created by default.

## Inside the crates

`api/src` holds `main.rs`, `app.rs`, `state.rs`, `error.rs`, plus `routes/`, `middleware/`, `handlers/`, and `dtos/`.
`domain/src` holds `lib.rs`, `errors.rs`, plus `entities/`, `use_cases/`, and `ports/`.
`infra/persistence` holds the Firestore client, the path helper, repositories, and seeds.
`infra/secrets` holds the vault reader and its cache. It implements the secret port.

## How files are cut

- One action per file. The file name is the action.
- `mod.rs` only declares and re-exports.
- A repository that grows splits into the struct, the reads, and the writes. Three files, one type.
- Tests live in `*_test.rs` or the crate's `tests/` directory.

## Contract copy

`AI_CONTEXT/` in a project receives `README.md` and `AGENTS.md` from the satori root, the files in `satori/backend/`, and a filled `CURRENT_TASK.md`.
