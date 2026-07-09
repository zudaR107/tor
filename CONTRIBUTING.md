# Contributing to Tor

Thanks for considering a contribution. Tor is just a Caddyfile and a
docker-compose.yml fronting the other three Schloss services — please keep
changes focused.

## Getting set up

See the [README](README.md) for running the whole platform locally.

## Before opening a PR

- Run `docker compose config` from this repo (with the sibling repos
  checked out alongside it) to confirm the compose file still resolves.
- If you changed the Caddyfile, validate it:
  `docker run --rm -v "$PWD/Caddyfile:/etc/caddy/Caddyfile:ro" caddy:2-alpine caddy validate --config /etc/caddy/Caddyfile`
- Keep commits focused; one logical change per PR is easier to review than
  several bundled together.
- Write commit messages that explain *why*, not just *what* — the diff
  already shows what changed.

## Opening a PR

- Branch from `main`.
- Reference the issue you're addressing if one exists (`Closes #123`).

## Reporting bugs / security issues

Open a regular issue for bugs. For anything that looks like a security
vulnerability, please use GitHub's private "Report a vulnerability" flow
under this repo's Security tab instead of a public issue.
