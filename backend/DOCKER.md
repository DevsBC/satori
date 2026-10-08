# Docker

    docker/
      dev/api.Dockerfile
      runtime/api.Dockerfile
      docker-compose.yml
    dev.ps1

## Port

Publish `8080`. The process reads `PORT` and defaults to `8080`. See `ENDPOINTS.md`.

## Dev (`docker/dev/api.Dockerfile`)

- debian-slim.
- Mounts the source.
- Hot reload with cargo-watch.
- No Firestore emulator.
- Connects to the dev GCP project.
- Context `"1"` is rejected by the process, same as `cargo run`.

## Runtime (`docker/runtime/api.Dockerfile`)

- Distroless when the binary has no extra native deps.
- A light distro when it needs them (ffmpeg and similar).
- Multi-stage build with `--features deployed`. The dev image and `cargo run` do not pass that feature. See `ENDPOINTS.md`.
- The runtime image contains the binary, not the source.
- Non-root user.
- No shell in the image when it is distroless.
- No secrets in the image.
- Compose healthcheck hits `GET /health` on port `8080`.

## Scripts and Docker

- `dev.ps1` and `dev.sh` run the API on the host machine (`cargo run -p api`). They are for a local machine, not for a cloud agent.
- Docker (`docker compose -f docker/docker-compose.yml`) is for a cloud agent, and for a local machine that has Docker.
- `docker/dev` is that local-or-cloud dev container. It rejects context `"1"`.
- `docker/runtime` is the deployed image.
- `.env` and `.env.example` only name `GOOGLE_APPLICATION_CREDENTIALS` and `GCP_PROJECT_ID`. No real secret in the example.

## Local context

Dev compose is a local process. It must not be given a way to call context `"1"`.
