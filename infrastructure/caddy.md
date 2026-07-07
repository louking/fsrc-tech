# Caddy (edge reverse proxy)

[Caddy](https://caddyserver.com/) is the edge reverse proxy sitting in front of all the dockerized apps and the website, both on the production server and for local development. It's distinct from the per-app Nginx + Gunicorn stacks some apps run internally (see e.g. [membertility](../applications/membertility.md)) — Caddy is the outermost layer that routes each domain to the right app container.

- Repo: https://github.com/louking/caddy-docker
- Repo's own `CLAUDE.md` (authoritative, deeper detail than this page): https://github.com/louking/caddy-docker/blob/main/CLAUDE.md

## Stack

- **caddy** — custom build (via `xcaddy`) with three extra modules: `transform-encoder` (structured logging), `caddy-dns/cloudflare` (DNS-01 TLS challenge for wildcard/non-HTTP certs), and `caddy-cloudflare-ip` (auto-refreshed Cloudflare IP ranges for `trusted_proxies`). Runs as a non-root user.
- **certbot** — obtains/renews a Let's Encrypt certificate for the DigitalOcean CDN custom domain (`cdn.steeplechasers.org`) via a Cloudflare DNS-01 challenge, and uploads it to the CDN through DigitalOcean's API. Checks for renewal every 12 hours.

## Configuration

- `config/Caddyfile` — active config, used for **development**: proxies `*.localhost` domains (e.g. `members.localhost`, `routes.localhost`, `scores.localhost`, `contracts.localhost`, `tmsim.localhost`, `logmon.localhost`) to the corresponding app's port on `host.docker.internal`.
- `config/Caddyfile.example` — production template with anonymized domains and Cloudflare TLS.
- The actual **production** Caddyfile lives directly on the server (`~/caddy-prod/config/Caddyfile`) and is managed there, not deployed from the repo — it's diverged significantly from the example and hosts all the real domains (loutilities.com, steeplechasers.org, scoretility.com, etc.).

## Deployment

Deployed with [Fabric](https://www.fabfile.org/): `fab -H <server-hostname> deploy prod` pulls `docker-compose.yml` from GitHub and runs `docker compose pull && docker compose up -d` on the server.

## Gotchas worth knowing

- HTTP/3 (QUIC) is explicitly disabled (`protocols h1 h2`) — Caddy advertises it by default via `Alt-Svc`, which breaks clients whose UDP/443 is blocked (`ERR_QUIC_PROTOCOL_ERROR`). Cloudflare already terminates HTTP/3 at its edge, so Caddy doesn't need to.
- `CF_API_TOKEN` is shared between the caddy service (DNS-based TLS) and the certbot service (CDN cert DNS challenge) — no separate token needed.

For full detail (all env vars, first-time CDN cert setup, debug mode), see the repo's own `CLAUDE.md` linked above.
