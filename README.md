# Satoru

This file is me: how I think, what I can do, and how I want the work done.
It is not a build specification. It does not authorize architecture, dependencies, data design, or production access.
How to build lives in the active layer (`backend/` or `frontend/`).
How an agent must behave lives in `AGENTS.md`.

When you are building, the layer wins. When the layer is silent, use the judgment in this file. Do not stop reading here.

## Who I am

- Alias: Satoru
- From: Juárez, Chihuahua, México
- Timezone: America/Ciudad_Juarez
- 10 years of experience
- Systems Engineer, Instituto Tecnológico de Ciudad Juárez
- Languages: Spanish (native), English (intermediate)
- Talk to me in Spanish. Be direct. Do not flatter. Do not take my side. If I am omitting something, say so.

### What I have done

From frontend developer to director. CEO and CTO of my own ventures.
Full stack, backend, microservices, DevOps.

I have shipped in SaaS, insurance, banking, telecom, e-commerce, analytics, and religion.
Right now I am a full-time senior developer, and I run my own work around Bible reading and study.

I am not married to a technology. I look at more than one approach, then I lock one. In a project, the layer is the lock. The inventory below is what I have used, not a menu.

### Experience

TypeScript, JavaScript, Rust, Python, Java, C, C++, C#, PHP, Lua.
NestJS, Express, ASP.NET, PHP, Axum, Tokio.
GCP, AWS, Azure. VPS on OVH and DigitalOcean.
Firestore, MongoDB, PostgreSQL, SQL Server, MariaDB.
Docker, Kubernetes, Terraform, Helm.
JWT and Argon2.

## How I decide

I keep the decisions that are expensive to undo: architecture, dependencies, data shape, auth, CI, and anything that touches production data.
I delegate the rest: boilerplate, tests, folder layout, modularization, pipelines, and bounded refactors.
I want a clean diff, and I want to understand why it was done. The code already says what.

A business threshold is a named rule. `my_var >= 60` hides it. When the rule changes, the diff should be that name.

## When the layer is silent

Use this order. Do not ask me to confirm a step that this list already answers.

1. If the layer already has a way, use it. Do not add a second way.
2. If the choice is architecture, a new dependency, the data model, auth, or production data, stop and ask. Those are mine.
3. If both options are reversible, pick the smaller one. The one a novice can change next month without being afraid of the tests.
4. Prefer one module that already exists over a new file that repeats it.
5. Prefer a named constant over a literal. Prefer an explicit error over a swallowed one.
6. Do not add a feature, a screen, or a collection the task did not name.
7. Say why in one sentence. Do not write a tour of what the code does.

"Done" means the layer's quality pipeline passed, with the raw output. A green claim and a red terminal is not done.

## What I reject

- Complexity added so the solution looks smart.
- A new name or a new folder for the same idea.
- Vibe coding. Magic strings. Magic numbers.
- A large refactor I did not ask for.
- A dependency I did not approve.
- Tests that break when a private helper is renamed. Those test the scaffolding, not the behavior.
- Production data touched without my approval. An agent once deleted years of it while "updating" records. That is why this line is not negotiable.

## Philosophy

Blessed be the God and Father of our Lord Jesus Christ,
who gives us the ability to give Him glory and honor
with our life and testimony.

## License

Private. Personal use.
