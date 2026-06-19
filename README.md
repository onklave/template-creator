# template-creator

The source-of-truth for **Onklave project template repositories** — the
standards, the build/deploy contract, and the step-by-step for authoring a new
template so every generated project starts from the same shape.

Onklave's project-onboarding wizard creates a greenfield repo by generating from
a GitHub *template repository* (`POST /repos/:owner/:repo/generate`). The catalog
of templates lives in the platform at
`_apps/@onklave/services/teams/src/project/templates/project-templates.ts`; each
entry's `templateRepo` points at one of the `onklave/template-*` repos these
standards govern.

## What's here

- **[STANDARDS.md](STANDARDS.md)** — the conventions every template repo follows
  (structure, the deploy contract, READMEs, containers, security).
- **[CHECKLIST.md](CHECKLIST.md)** — how to add a brand-new template, end to end.
- **[AGENTS.md](AGENTS.md)** — the brief for an agent scaffolding a template repo.
- **[baseline/](baseline)** — drop-in baseline files (`.editorconfig`,
  `.dockerignore`, `.gitignore` fragments, a README skeleton) to copy into a new
  template.

## The contract, in one breath

A template is a **runnable, idiomatic, minimal** skeleton for one stack that
honours its catalog `deployPreset`:

- **Service** (`runtime: node | container`, or any stack with a port): a
  `Dockerfile` at the repo root, a multi-stage build, runs **non-root**, listens
  on `PORT` (default per preset), and serves **`GET /healthz` → 200**.
- **Worker** (`runtime: python`, no port): a `Dockerfile`, non-root, a run loop
  with **graceful SIGTERM/SIGINT shutdown**. No inbound port, no health route.
- **Static** (`runtime: static`): no server — `npm run build` produces `dist/`,
  the artifact Onklave serves from the edge. No Dockerfile, no port.

If a template's Dockerfile/port/health don't match its `deployPreset`, the
Deployments tab will mislead — keep them in lockstep.

## The existing templates

| Repo | Catalog id | Stack |
|---|---|---|
| `template-node-web-service` | `node-web-service` | Node.js HTTP service (Express) |
| `template-static-site` | `static-site` | Vite + TypeScript static site |
| `template-python-worker` | `python-worker` | Python background worker |
| `template-container` | `container` | Go + distroless (BYO-Dockerfile starter) |
