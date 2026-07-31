# tor

[![Test](https://github.com/zudaR107/tor/actions/workflows/test.yml/badge.svg)](https://github.com/zudaR107/tor/actions/workflows/test.yml)
[![License: AGPL v3](https://img.shields.io/badge/license-AGPL--3.0-blue.svg)](LICENSE)

tor ("gate" in German) is the single-entrypoint reverse-proxy gateway for
the Hof platform. It fronts every service by subdomain so nobody needs to
remember or type a port.

Part of the [Hof platform](https://github.com/zudaR107/Hof) — a suite of
self-hosted personal services:

- [`schloss`](https://github.com/zudaR107/schloss) — home page / launcher
- [`schlussel`](https://github.com/zudaR107/schlussel) — auth: accounts, login, tokens
- [`kuvert`](https://github.com/zudaR107/kuvert) — envelope budgeting
- [`tafel`](https://github.com/zudaR107/tafel) — task/project tracking
- **`tor`** (this repo) — reverse-proxy gateway all of the above sit behind
- [`schloss-ui`](https://github.com/zudaR107/schloss-ui) — shared frontend components
- [`schloss-server-kit`](https://github.com/zudaR107/schloss-server-kit) — shared backend auth/CORS kit

## How it fits into the platform

tor ships no application code of its own — just a Caddyfile and a
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

- `https://localhost` — Schloss (home)
- `https://auth.localhost` — Schlüssel (login/register)
- `https://kuvert.localhost` — Kuvert

`*.localhost` resolves to `127.0.0.1` automatically in every modern browser
— no `/etc/hosts` editing needed. Caddy auto-upgrades these to HTTPS; since
`localhost` can't get a real Let's Encrypt certificate, it signs them with
its own local CA instead (see below - one-time browser setup needed).

### Trusting the local HTTPS certificate (one-time, local dev only)

The gateway's certs for `*.localhost` are signed by a CA Caddy generates
itself and stores in the `caddy-data` volume - your browser doesn't know
about it yet, so the first visit shows a certificate warning
(`SEC_ERROR_UNKNOWN_ISSUER` in Firefox, "Not secure" in Chrome). Trust it
once per machine:

```sh
docker compose exec gateway \
  cat /data/caddy/pki/authorities/local/root.crt > caddy-local-root.crt
```

**Firefox**: `about:preferences#privacy` → scroll to Certificates → *View
Certificates* → *Authorities* tab → *Import…* → select `caddy-local-root.crt`
→ check *"Trust this CA to identify websites"* → OK. Restart Firefox.

**Chrome/system trust store** (Linux): `sudo cp caddy-local-root.crt
/usr/local/share/ca-certificates/caddy-local-root.crt.crt && sudo
update-ca-certificates` (Debian/Ubuntu; other distros use their own
equivalent), then restart the browser. macOS: import into Keychain Access
(System keychain, "Always Trust"). Windows: import via `certutil -addstore
-f "ROOT" caddy-local-root.crt` or the Certificates MMC snap-in.

This only needs to happen again if the `caddy-data` volume is ever removed
(e.g. `docker compose down -v`, or `docker volume rm caddy-data`) - that
regenerates the CA with a new key, and the old trusted entry needs
replacing (delete the old "Caddy Local Authority" entry first, then import
the new `root.crt`, or you'll see `SEC_ERROR_BAD_SIGNATURE` instead of the
usual unknown-issuer warning).

### Running a single service standalone

Each repo's own `docker-compose.yml` still works on its own for isolated
development of just that service (still needs the shared `schloss-net`
network created once, and the other services it depends on already
running, same as before tor existed).

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
