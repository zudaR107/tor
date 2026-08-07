# tor

[![Test](https://github.com/zudaR107/tor/actions/workflows/test.yml/badge.svg)](https://github.com/zudaR107/tor/actions/workflows/test.yml)
[![License: AGPL v3](https://img.shields.io/badge/license-AGPL--3.0-blue.svg)](LICENSE)

Part of the [Hof platform](https://github.com/zudaR107/Hof) — a suite of
self-hosted personal services:

- [`schloss`](https://github.com/zudaR107/schloss) — home page / launcher
- [`schlussel`](https://github.com/zudaR107/schlussel) — auth: accounts, login, tokens
- [`kuvert`](https://github.com/zudaR107/kuvert) — envelope budgeting
- [`tafel`](https://github.com/zudaR107/tafel) — task/project tracking
- [`zettel`](https://github.com/zudaR107/zettel) — markdown note-taking
- **`tor`** (this repo) — reverse-proxy gateway all of the above sit behind
- [`schloss-ui`](https://github.com/zudaR107/schloss-ui) — shared frontend components
- [`schloss-server-kit`](https://github.com/zudaR107/schloss-server-kit) — shared backend auth/CORS kit

tor ("gate" in German) is the single-entrypoint reverse-proxy gateway for
the Hof platform. It fronts every service by subdomain so nobody needs to
remember or type a port.

## How it fits into the platform

tor ships no application code of its own — just a Caddyfile and a
docker-compose.yml. It routes by `Host:` header to each app's web-facing
Compose service on internal port `80`. The current routes are:

| Public host | Compose service |
|---|---|
| `{$DOMAIN}` | `schloss` |
| `auth.{$DOMAIN}` | `schlussel-frontend` |
| `kuvert.{$DOMAIN}` | `kuvert-frontend` |
| `tafel.{$DOMAIN}` | `tafel-frontend` |
| `zettel.{$DOMAIN}` | `zettel-frontend` |

The API services (`schlussel`, `kuvert-backend`, `tafel-backend`, and
`zettel-backend`) remain internal dependencies and are not direct gateway
targets.

## Running the whole platform

Assumes the standard layout: `schlussel/`, `schloss/`, `kuvert/`, `tafel/`,
`zettel/`, and `tor/` as sibling directories (this is exactly what you get
by cloning [`Hof`](https://github.com/zudaR107/Hof) with
`--recurse-submodules`, or cloning all six repos into the same parent
folder by hand).

```sh
docker network create schloss-net   # one-time
cp .env.example .env
docker compose up -d --build
```

That's it — this one command starts all five apps plus the gateway, via
`include:` pulling in each sibling repo's own `docker-compose.yml`. In
Compose terms that is nine application services plus `gateway`, because
four apps have separate backend and frontend services.

- `https://localhost` — Schloss (home)
- `https://auth.localhost` — Schlüssel (login/register)
- `https://kuvert.localhost` — Kuvert
- `https://tafel.localhost` — Tafel
- `https://zettel.localhost` — Zettel

`*.localhost` resolves to `127.0.0.1` automatically in every modern browser
— no `/etc/hosts` editing needed. Caddy auto-upgrades these to HTTPS; since
`localhost` can't get a real Let's Encrypt certificate, it signs them with
its own local CA instead (see below - one-time browser setup needed).

An unknown local app host such as `https://typo.localhost/path` receives its
own exact-hostname certificate from that local CA and redirects to
`https://localhost/path`. The gateway does not issue an internal-CA
certificate for an unknown production host such as `typo.example.com`:
the on-demand TLS policy denies it, so the TLS handshake fails before an
HTTP redirect or response can be sent. This prevents a production gateway
from acting as an internal-certificate oracle for arbitrary hostnames.

### Trusting the local HTTPS certificate (one-time, local dev only)

The gateway's certificates for its named localhost sites, and exact
certificates issued for unknown `*.localhost` hosts, are signed by a CA
Caddy generates and stores in the `caddy-data` volume - your browser
doesn't know about it yet, so the first visit shows a certificate warning
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
`auth.<domain>`, `kuvert.<domain>`, `tafel.<domain>`, and
`zettel.<domain>`) at this host - see `.env.production.example` for a
filled-in starting point:

```sh
cp .env.production.example .env   # then edit DOMAIN to your real domain
docker compose up -d --build
```

Caddy provisions HTTPS automatically via Let's Encrypt for each subdomain —
no certificate setup required. The local unknown-host redirect is
deliberately limited to `*.localhost`; an unknown host under the production
domain is denied during TLS setup and does not receive Caddy's internal
certificate.

### Environment variables

See `.env.example` — one file covers every variable needed by any of the
five included services, since `include:` shares one Compose project
environment. The important one is `DOMAIN`; the rest are origin/CORS
allowlists and cross-service URLs that already default to the matching
`*.localhost` subdomain scheme.

## License

AGPL-3.0 — see [LICENSE](LICENSE).
