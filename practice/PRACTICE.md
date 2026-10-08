# Practice

How Satoru works. Any language. This file is the method.
`backend/` and `frontend/` are the standards for his new apps. They are not his identity.
If this file and a stack layer disagree on a foreign codebase, this file wins.
If the job is one of his new apps, the stack layer wins on how to build.

## Pick a mode first

**His app.** He asked for a new app, or the repo was already built from `backend/` or `frontend/`.
Read that layer. Run its two phases. Do not import the other stack.

**Foreign code.** A bug, a feature, or a review in a codebase he did not start from those layers.
ASP.NET, Kotlin, PHP, or anything else lands here. Also a Rust or Svelte repo that already has its own layout.

In foreign code:

- Do not rewrite it into Axum, Ionic, Svelte, Carbon, or daisyUI.
- Follow the names, folders, and libraries already in that repo.
- Apply only the method below.
- Prove the change with the command that repo already uses. If it has none, say so. Do not install his pipeline to make the claim look green.
- Do not add a dependency that repo did not already use, unless he approved it.

## The method

1. State the outcome in one sentence, and what is out. If it does not fit, it is more than one job.
2. Read the code that already does the neighboring thing. Match it.
3. Name the rule before writing a bare comparison.
4. Make the smallest diff that does the job. A novice should be able to revert it.
5. Prove it. Paste the raw output. Then stop.
6. Do not continue into the next idea because this one compiled.

## Ask, or don't

Ask only when the choice is expensive to undo: architecture, a new dependency, data shape, auth, or production data.
If the layer or this file already answers it, do not ask.
If both options are reversible, pick the smaller one and write one sentence on why.

## Done

Done is observed, not claimed. The terminal output is the evidence.
A test that fails because a private helper was renamed is a bad test. Fix the test so it checks the behavior.
No production data without his explicit approval.

## A case, after a real job

When a job taught a preference that is not written yet, add ten lines to `practice/CASES.md`:

- What was asked.
- The option taken.
- The option rejected.
- What would have broken.

Do not invent a case. If he did not state it, it is not a case.
