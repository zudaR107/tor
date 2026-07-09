# Tor

[![Test](https://github.com/zudaR107/tor/actions/workflows/test.yml/badge.svg)](https://github.com/zudaR107/tor/actions/workflows/test.yml)
[![License: AGPL v3](https://img.shields.io/badge/license-AGPL--3.0-blue.svg)](LICENSE)

Tor ("gate" in German) is the single-entrypoint reverse-proxy gateway for
the Schloss platform. It fronts all three services by subdomain so nobody
needs to remember or type a port.

Part of the [Schloss platform](https://github.com/zudaR107/Hof) — see that
repo for how all four services fit together.

## How it fits into the platform

Each service is its own repo, named after a German word related to what it
does:

- [`schloss`](https://github.com/zudaR107/schloss) — the home page / launcher
- [`schlussel`](https://github.com/zudaR107/schlussel) — auth: accounts, login, tokens
- [`kuvert`](https://github.com/zudaR107/kuvert) — envelope budgeting
- **`tor`** (this repo) — the gateway all of the above sit behind

Tor ships no application code of its own — just a Caddyfile and a
docker-compose.yml. It routes by `Host:` header to each service's own
existing Caddy-fronted web image (each already serves from `/` on its
internal port `80`), so none of the other three repos needed any code
changes to work behind it.

## Running the whole platform

Assumes the standard layout: `schlussel/`, `schloss/`, `kuvert/`, and `tor/`
as sibling directories (this is exactly what you get by cloning
[`Hof`](https://github.com/zudaR107/Hof) with `--recurse-submodules`, or
cloning all four repos into the same parent folder by hand).

```sh
docker network create schloss-net   # one-time
cp .env.example .env
docker compose up -d --build
```

That's it — this one command starts all three services plus the gateway,
via `include:` pulling in each sibling repo's own `docker-compose.yml`.

- `http://localhost` — Schloss (home)
- `http://auth.localhost` — Schlüssel (login/register)
- `http://kuvert.localhost` — Kuvert

`*.localhost` resolves to `127.0.0.1` automatically in every modern browser
— no `/etc/hosts` editing needed.

### Running a single service standalone

Each repo's own `docker-compose.yml` still works on its own for isolated
development of just that service (still needs the shared `schloss-net`
network created once, and the other services it depends on already
running, same as before Tor existed).

## Production

Set `DOMAIN` to a real domain you control, and point its DNS (plus
`auth.<domain>` and `kuvert.<domain>`) at this host - see
`.env.production.example` for a filled-in starting point:

```sh
cp .env.production.example .env   # then edit DOMAIN to your real domain
docker compose up -d --build
```

Caddy provisions HTTPS automatically via Let's Encrypt for each subdomain —
no certificate setup required.

### Environment variables

See `.env.example` — one file covers every variable needed by any of the
four included services, since `include:` shares one Compose project
environment. The important one is `DOMAIN`; the rest are origin/CORS
allowlists and cross-service URLs that already default to the matching
`*.localhost` subdomain scheme.

## License

AGPL-3.0 — see [LICENSE](LICENSE).
