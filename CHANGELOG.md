# Changelog

Brief log of notable changes, grouped by theme — not a full commit history
(see `git log` for that). New entries get appended under the section they
fit best; add a new section if none fits.

## Gateway
- Initial subdomain-routing Caddy gateway fronting schloss, schlussel, and
  kuvert behind a single host port, with `include:`-based one-command
  startup for the whole platform.

## Docs
- Repo renamed from `Tor` to `tor` for slug consistency with the other
  three services.
- License/CI badges, a link to the Hof meta-repo, `.env.production.example`
  for real-domain deployment, and a missing `.gitignore` for `.env`.
