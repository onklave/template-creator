# Adding a new Onklave project template

End-to-end checklist. Read [STANDARDS.md](STANDARDS.md) first.

## 1. Decide the shape
- [ ] Pick the catalog `id` (kebab-case, e.g. `go-web-service`) and the repo name
      `template-<id>`.
- [ ] Pick the `runtime` and `deployPreset` (dockerfile? port? `/healthz`?
      static `dist/`?). This is the contract the repo must honour.

## 2. Create the repo
- [ ] `gh repo create onklave/template-<id> --public --add-readme --description "..."`
- [ ] It will be marked a template after scaffolding (step 4).

## 3. Scaffold it
- [ ] Copy the relevant `baseline/` files (`.editorconfig`, `.dockerignore`,
      `.gitignore`, README skeleton) as a starting point.
- [ ] Build the minimal idiomatic skeleton per [STANDARDS.md](STANDARDS.md):
      source, one real test, a multi-stage non-root `Dockerfile` (service/worker)
      or a `build → dist/` (static), and a README following §6.
- [ ] Keep the Dockerfile/port/health in lockstep with the `deployPreset`.
- [ ] Verify from a clean checkout: build green, tests pass, `docker build .`
      succeeds (service/worker) or `npm run build` emits `dist/` (static).

## 4. Mark it a template
- [ ] `gh api -X PATCH repos/onklave/template-<id> -f is_template=true`
- [ ] Confirm: `gh api repos/onklave/template-<id> --jq .is_template` → `true`.

## 5. Wire the catalog (platform repo)
In `_apps/@onklave/services/teams/src/project/templates/project-templates.ts`:
- [ ] Add the `ProjectTemplateDef` with the `deployPreset`, an inline `scaffold`
      fallback (minimal), and `templateRepo: { repo: 'template-<id>' }`.
- [ ] Build the teams service; open the change as part of the next batch.

## 6. Verify the loop
- [ ] In the onboarding wizard, pick the new template + "create new repo" and
      confirm a generated repo comes from the template (full tree, not the inline
      fallback) and the Deployments tab shows the matching deploy pattern.

> Tip: an agent can do steps 3–4 from a one-paragraph brief — see [AGENTS.md](AGENTS.md).
