# metamcp

Deploy config for [MetaMCP](https://github.com/metatool-ai/metamcp) — MCP
aggregator/orchestrator/gateway. Standalone-usable with plain Docker Compose (bundles its
own Postgres, matching upstream's default), or deployed on Coolify connected to this
tenant's shared Postgres instance.

## Standalone

```bash
cp .env.example .env
# set BETTER_AUTH_SECRET: openssl rand -hex 32
docker compose up -d
# UI at http://localhost:12008
```

## On Coolify

Deployed with `docker_compose_location` set to `/docker-compose.coolify.yaml`. That variant
does not bundle Postgres — it expects a `postgresql` service reachable on the `coolify`
network (see the `postgresql` repo) with a `metamcp` database and dedicated user already
created. `POSTGRES_USER`/`POSTGRES_PASSWORD`/`BETTER_AUTH_SECRET`/`APP_URL` are all set as
Coolify environment variables, not committed here — all four are required with no default,
so a missing one fails the deploy loudly rather than silently booting misconfigured.
`APP_URL` specifically must be the real domain for whichever tenant is deploying this repo
(e.g. `https://mcp.jdmnt.co`) — never hardcode a domain into `docker-compose.coolify.yaml`
itself, that would defeat the point of this being a reusable, tenant-agnostic repo.
