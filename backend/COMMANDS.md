# COMMANDS.md

The pass/fail rules are in `QUALITY.md`. This file is only the commands.

## Quality pipeline

    cargo fmt --all -- --check
    cargo clippy --workspace --all-targets --all-features -- -D warnings
    cargo test --workspace

PowerShell, doc step:

    $env:RUSTDOCFLAGS = "-D warnings"; cargo doc --workspace --no-deps

bash, doc step:

    RUSTDOCFLAGS="-D warnings" cargo doc --workspace --no-deps

## Dev on the host

`dev.ps1` and `dev.sh` run this. They do not start Docker.

    cargo run -p api

The process listens on `PORT` or `8080`. It rejects context `"1"`.

## Docker

For a cloud agent, or for a local machine that has Docker.

    docker compose -f docker/docker-compose.yml up
    docker build -f docker/runtime/api.Dockerfile -t app .

## Build

    cargo build --release

## 150-line check

PowerShell, from the repo root:

    Get-ChildItem -Recurse -Filter *.rs -Path api, domain, infra |
      Where-Object { $_.FullName -notmatch '\\target\\' } |
      ForEach-Object {
        $n = @(Get-Content -LiteralPath $_.FullName).Count
        if ($n -gt 150) { '{0} {1}' -f $n, $_.FullName }
      }

bash:

    find api domain infra -name '*.rs' -not -path '*/target/*' | while read -r f; do
      n=$(wc -l < "$f")
      if [ "$n" -gt 150 ]; then echo "$n $f"; fi
    done

Empty output means pass.

## Docs in a browser

The pipeline already ran `cargo doc`. OpenAPI in a browser needs the api up: `http://localhost:8080/docs` and `http://localhost:8080/api-docs/openapi.json`. That visit is not a gate.
