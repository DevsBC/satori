# QUALITY.md — Validation contract

## Rule

A task is NOT done until the quality pipeline passes.
No exceptions. No "mostly passes". No "only warnings".

## Pipeline (must all pass)

    cargo fmt --all -- --check
    cargo clippy --workspace --all-targets --all-features -- -D warnings
    cargo test --workspace
    cargo doc --workspace --no-deps

Commands you can paste are also listed in `COMMANDS.md`. The definition of pass is this file.

## Zero warnings

- Clippy warnings are errors (`-D warnings`).
- Rustc warnings are errors in CI (`RUSTFLAGS=-D warnings`).
- Rustdoc warnings are errors (`RUSTDOCFLAGS=-D warnings`) on the doc step.
- If you introduce a warning, you fix it before claiming done.
- If a warning is unavoidable, stop and ask. An `#[allow]` needs operator approval and a comment that says why.

## Order of execution

1. `cargo fmt --all -- --check`
2. `cargo clippy --workspace --all-targets --all-features -- -D warnings`
3. `cargo test --workspace`
4. `cargo doc --workspace --no-deps` with `RUSTDOCFLAGS=-D warnings`

If a step fails, fix it. Do not skip ahead. After a fix, re-run from step 1.

Step 3 is the OpenAPI gate. The `api` test that serializes the spec runs inside `cargo test`. There is no extra OpenAPI command, because a second command gets forgotten. The test does not start a server.

Step 4 is the docs gate. Broken doc links and broken examples fail the build. A missing `///` does not fail it. Comments are for why, not a quota. See `BACKEND.md`.

What step 3 does not prove: that every Axum route is in the spec. That is why the only router is an `OpenApiRouter` (`ENDPOINTS.md`). A route mounted beside it is a contract break even if the test is green.

## Evidence required

Paste the raw output of each command.
No summaries. No "it passed". Raw output.

## 150-line limit

- CI fails if any `.rs` file under `api/`, `domain/`, or `infra/` exceeds 150 lines.
- Count is every line in the file, including blanks and comments.
- Exclude `target/`.
- Test modules in the same file count toward the 150. Prefer `tests/` or `*_test.rs` when the implementation is near the cap.
- `mod.rs` only declares and re-exports.
- The shell check is in `COMMANDS.md`.

## Tests

`cargo test --workspace` passes with no network and no GCP credentials.

A test calls a public use case and asserts the result. It does not call a private function, and it does not assert how many times a port was called. Renaming an internal helper must not break a test. If it does, the test was tied to the implementation, not to the behavior.

One use case, one `*_test.rs` file next to it. One test, one behavior. The name is the behavior: `signup_rejects_a_duplicate_email`. No `test1`, no file per assertion.

Fakes live in one module, `domain` test support, shared by every test. A fake is a struct that implements the port and keeps data in memory. It is not a mock. No crate that records expectations (`mockall` and the like). A new fake per test is a failure of this rule.

Do not copy behaviors into this file. The behavior lives in the layer (`AUTH.md`, `ENDPOINTS.md`, and the product task). A test exists only when one of those rules can be broken without a compile error. One rule, one test. No second test for the same rule. Phase 2 adds tests only for rules the task added. It does not retest phase 1.

A test that opens Firestore is `#[ignore]`, context `"0"` only, and it deletes only ids that same test created. It is not part of this pipeline.

Why the fake is in memory: the use case is the real code. Firestore is the adapter. A novice can change the adapter and the behavior tests stay green. A live test against a shared project is not a fixture.

## Additional gates

- The OpenAPI document serializes (the `api` test in `ENDPOINTS.md`).
- No secrets in the diff.
- No `unwrap()` or `expect()` in `infra` or in `api` handlers unless the operator approved it and the line says why.
- No `println!` in library or binary code. Use `tracing`.

## Failure protocol

If any gate fails:

1. Stop.
2. Report the raw failure.
3. Propose a fix.
4. Do not proceed to the next gate.
5. Do not claim done.
