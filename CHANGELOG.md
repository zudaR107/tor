# Changelog

Brief log of notable changes, grouped by theme — not a full commit history
(see `git log` for that). New entries get appended under the section they
fit best; add a new section if none fits.

## Gateway
- Initial subdomain-routing Caddy gateway fronting schloss, schlussel, and
  kuvert behind a single host port, with `include:`-based one-command
  startup for the whole platform.
- Fixed `.env.example`'s origin defaults (`ALLOWED_ORIGINS`,
  `ALLOWED_RETURN_ORIGINS`, `DEFAULT_APP_URL`, `KUVERT_URL`,
  `SCHLUSSEL_WEB_URL`) from `http://` to `https://` - Caddy's automatic
  HTTPS upgrades every request here, so the old defaults failed the
  return_to allowlist for anyone copying this file as-is.
- Split `ALLOWED_ORIGINS` into `SCHLUSSEL_ALLOWED_ORIGINS` and
  `KUVERT_ALLOWED_ORIGINS` in `.env.example`/`.env.production.example` -
  one shared name fed the same value into both schlussel and kuvert-api's
  compose files, silently overriding kuvert-api's own CORS allowlist.
  Requires schlussel#46 and kuvert#59's matching compose var renames.
- Added `SCHLOSS_URL` to `.env.example`/`.env.production.example` -
  kuvert's new header needs a URL to link back to schloss's home page,
  which no existing var covered (the others all point the other way).
- Fixed the TLS handshake failing outright (`SSL_ERROR_INTERNAL_ERROR_ALERT`
  in Firefox) on any subdomain that isn't one of the three declared sites
  (e.g. a typo'd `schlussel.localhost`) - Caddy only eagerly provisions
  certs for the three declared hostnames, so it had nothing to present for
  an unrecognized SNI and aborted the handshake before any HTTP routing
  could happen. Added a catch-all `*.{$DOMAIN}` site with `tls internal`
  (one wildcard cert, same local CA as the other three sites) redirecting
  to the main site.

## Docs
- Repo slug renamed to lowercase for consistency with the other three
  services - the project name is now written lowercase everywhere,
  including at the start of sentences.
- License/CI badges, a link to the Hof meta-repo, `.env.production.example`
  for real-domain deployment, and a missing `.gitignore` for `.env`.
- Fixed remaining prose mentions of the project name to lowercase "tor"
  throughout README.md and CONTRIBUTING.md, including at sentence starts.
- Added CODE_OF_CONDUCT.md, SECURITY.md, issue templates, and a pull
  request template.
- Documented the one-time step of trusting Caddy's local HTTPS CA in the
  browser for local dev (`*.localhost` can't get a real Let's Encrypt cert),
  including how to re-trust it if `caddy-data` ever gets wiped and the CA
  regenerates with a new key (previously undocumented, discovered when a
  wiped volume produced a `SEC_ERROR_BAD_SIGNATURE` instead of the usual
  unknown-issuer warning). Fixed the README's example URLs from `http://` to
  `https://` to match Caddy's actual automatic upgrade behavior.
