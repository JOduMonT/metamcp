# CLAUDE.md — metamcp

Deploy config for [MetaMCP](https://github.com/metatool-ai/metamcp), adapted from upstream's
own `docker-compose.yml`. Read `README.md` first for the standalone-vs-Coolify basics — this
file is the "don't repeat past mistakes" layer.

## This repo must stay tenant-agnostic

`docker-compose.coolify.yaml` must never contain a hardcoded domain, database name beyond
`metamcp` (that one's fixed by convention — every tenant gets a database named after the
app), or secret. Every tenant-specific value is a required env var
(`${VAR:?VAR must be set}`) supplied by whoever deploys this repo, via Coolify environment
variables — not committed here. That's what makes this repo genuinely reusable by more than
one tenant. `APP_URL`/`NEXT_PUBLIC_APP_URL` in particular were briefly hardcoded to one
tenant's domain during initial deployment and had to be fixed — don't reintroduce that.

## A `ports:` block in `docker-compose.coolify.yaml` is a real security incident, not a style
nit

This repo's own history: `docker-compose.coolify.yaml` briefly carried a `ports: -
"12008:12008"` block copied from the standalone file. That publishes the container directly
on the server's public IP in plaintext — no TLS, no Traefik, no Cloudflare. Confirmed live:
the app (with real user session cookies, API keys, and OAuth tokens in its database) was
reachable unencrypted at `http://<server-ip>:12008/` before this was caught and fixed.
Traefik reaches the container over the `coolify` network and auto-selects the image's single
`EXPOSE`d port (12008) — a host port mapping is never needed in the Coolify variant. The
standalone `docker-compose.yaml` (never deployed, local `docker compose up` only) is fine to
publish ports.

## Version pinning

Both compose files pin a concrete `ghcr.io/metatool-ai/metamcp` tag, never `:latest` — check
the upstream repo's releases for the current one before bumping
(`gh api repos/metatool-ai/metamcp/releases/latest`, or let Renovate open the PR).
`pull_policy: always` was removed once pinned — it's redundant and defeats the point of
pinning.

## Database

Connects to a shared Postgres instance (a separate `postgresql` repo/Coolify app) at
hostname `postgresql` on the `coolify` network — does not bundle its own Postgres in the
Coolify variant (the standalone variant does, matching upstream's default, so this repo
still works with zero other infrastructure for local use). The shared instance's database
and user must already exist with `GRANT ALL ON SCHEMA public` applied (a Postgres 15+
requirement most people miss) — see the Hub's `README.md` "Adding a tenant app" runbook if
you have access to it, or just know that a fresh `CREATE DATABASE`/`CREATE USER`/`GRANT ALL
PRIVILEGES ON DATABASE` is *not* enough on its own; migrations will fail with `permission
denied for schema public` and the container will crash-loop.
