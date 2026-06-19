# {{name}}

> One line: what this is. Scaffolded from an **Onklave project template**.

## Run locally
```
<the exact commands, e.g. npm ci && npm run dev>
```
Serves on `http://localhost:<port>` · health: `GET /healthz`.
<For a worker: how to run the loop. For static: npm run dev.>

## Test / build
```
<commands>
```

## How Onklave deploys it
<Service: built from the Dockerfile, served on port <port>, health-checked at /healthz.>
<Worker: built from the Dockerfile, run as a background worker (no port).>
<Static: `npm run build` → `dist/`, served from the edge (no server).>
