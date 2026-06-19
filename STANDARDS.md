# Onklave template standards

Every `onklave/template-*` repo MUST follow these. The goal: a developer who
generates a project from any template gets the *same* clean, idiomatic,
production-shaped skeleton — different stack, same standard.

## 1. Principle: solid but minimal

A template is a **starting point**, not a sample app. A senior engineer should
read it in two minutes and see a tidy, runnable skeleton — not a kitchen sink.
No demo CRUD, no unused dependencies, no framework unless it earns its place.
Prefer the standard library; add a dependency only when it removes real work.

## 2. The deploy contract

Each template maps to a catalog entry with a `deployPreset`. The repo MUST honour
it exactly, because that preset is what the Onklave Deployments surface uses.

| `runtime` | Build | Run | Health | Port |
|---|---|---|---|---|
| `node` / `container` (service) | `Dockerfile` at root, multi-stage | non-root | `GET /healthz` → 200 | reads `PORT`, default = preset port |
| `python` (worker) | `Dockerfile` at root, multi-stage | non-root | none (no port) | none |
| `static` | `npm run build` → `dist/` | served from edge (no server) | none | none |

Rules:
- A **service** listens on `process.env.PORT` (or the language equivalent),
  defaulting to the preset's port, and answers `GET /healthz` with 200 and a
  tiny body. Nothing else is the health route.
- A **worker** has no inbound port and installs a **graceful shutdown** handler
  (SIGTERM + SIGINT) that stops the loop and exits 0 — no traceback on Ctrl-C.
- A **static** site's deploy artifact is `dist/`; never commit it (gitignore it).

## 3. Repository layout

- Source under `src/` (or the language's idiom — `main.go` at root for Go is
  fine). Tests alongside or under `test/` / `tests/`.
- Exactly one `README.md` (see §6), `.gitignore`, and — for containerised
  templates — `Dockerfile` + `.dockerignore`, all at the repo root.
- `.editorconfig` at the root (copy `baseline/.editorconfig`).
- No build output, no `node_modules/`, no virtualenvs, no secrets committed.

## 4. Containers

- **Multi-stage** always: a build stage, then a slim/minimal final stage.
- Final image runs as a **non-root** user.
- Smallest sensible base: `node:22-alpine`, `python:3.12-slim`,
  `gcr.io/distroless/*` or `scratch` for compiled binaries.
- `EXPOSE` the service port; a `HEALTHCHECK` is encouraged (for distroless with
  no shell, make the binary self-probe rather than add curl/wget).
- A `.dockerignore` that excludes VCS, deps, tests, and local cruft.
- `docker build .` MUST succeed from a clean checkout.

## 5. Toolchain & tests

- Pin the language version (engines / `requires-python` / `go` directive) and
  pin it to the same version the Dockerfile uses.
- Provide a **lockfile** where the ecosystem has one (`package-lock.json`,
  etc.) so `npm ci` / reproducible installs work — both locally and in the
  Dockerfile.
- At least one **real test** that exercises the contract (the `/healthz`
  handler, the worker's unit of work, that the static build emits `dist/`).
- Scripts/targets for the common loop: run, build, test (and lint where it fits).
- A clean checkout MUST build green and pass tests with the documented commands.

## 6. README

Every template README explains, in this order:
1. **What it is** — one line, and that it's an *Onklave project template*.
2. **Run locally** — the exact commands (`npm ci && npm run dev`, etc.).
3. **Test / build** — the commands and what success looks like.
4. **How Onklave deploys it** — the contract: builds from the `Dockerfile` and
   serves on port N with `/healthz` (service), runs as a worker (no port), or
   serves `dist/` from the edge (static).
5. For the generic `container` template: that you may swap in **any** stack as
   long as you keep the contract (Dockerfile at root · listen on the port ·
   `/healthz` → 200).

## 7. Security & hygiene

- No secrets, tokens, or `.env` files with real values — provide `.env.example`
  if config is needed.
- Non-root containers; least-privilege.
- Keep dependencies current and advisory-clean (`npm audit` / `pip` / `govulncheck`
  with no known-vulnerable pins at authoring time).
- License/headers per the org default if the org requires them.

## 8. Catalog wiring (platform side)

A template repo is only reachable once the platform catalog references it:
- Add/confirm the entry in `project-templates.ts` with a matching `deployPreset`
  and `templateRepo: { repo: 'template-<id>' }` (owner defaults to `onklave`,
  override via `ONKLAVE_TEMPLATE_REPO_OWNER`).
- The repo MUST be marked a **Template repository** (`is_template: true`) and be
  reachable by the org's connected GitHub token (public is simplest).
- The repo's Dockerfile/port/health MUST match the preset (see §2).
