# Brief for an agent scaffolding a template repo

Use this as the template for the prompt you give a scaffolding agent. Fill in the
`<...>` placeholders. One agent per repo; agents are independent and run in
parallel safely (each works in its own temp clone).

---

You are scaffolding the GitHub template repository `onklave/template-<id>`. `gh`
is authenticated with admin on the onklave org — use it as-is.

**Work in isolation**: clone into a fresh temp dir and work ONLY there. Never
touch any other repo or working tree on the machine.

1. `gh repo clone onklave/template-<id> /tmp/scaffold-<id> && cd /tmp/scaffold-<id>`
2. Build a **solid but minimal**, idiomatic skeleton for `<stack>` honouring the
   deploy contract: `<runtime>`, `<dockerfile? / static dist/>`, port `<port>`,
   health `<GET /healthz / none>`. Follow the Onklave template standards:
   - Service: multi-stage non-root `Dockerfile` at root, listens on `PORT`
     (default `<port>`), `GET /healthz` → 200.
   - Worker: multi-stage non-root `Dockerfile`, graceful SIGTERM/SIGINT shutdown,
     no port/health.
   - Static: `npm run build` → `dist/` (gitignored), no Dockerfile/port.
   - Include: source, ONE real test exercising the contract, a lockfile if the
     ecosystem has one, `.gitignore`, `.editorconfig`, `.dockerignore` (containers),
     and a README per the standards (what it is + run + test/build + how Onklave
     deploys it + that it's an Onklave project template).
3. Verify from a clean checkout where tooling exists: build green + tests pass
   (and `docker build .` for containerised templates).
4. Commit on `main` and push:
   `git add -A && git commit -m "feat: scaffold <stack> starter" && git push origin main`
   End the commit body with: `Co-Authored-By: Claude <model> <noreply@anthropic.com>`
5. Mark it a template: `gh api -X PATCH repos/onklave/template-<id> -f is_template=true`
6. Report: files created, the test/build result, the push, and confirm
   `is_template` is `true`.

---

The canonical rules live in [STANDARDS.md](STANDARDS.md); the full sequence
(including catalog wiring) is in [CHECKLIST.md](CHECKLIST.md).
