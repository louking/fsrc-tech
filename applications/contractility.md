# contractility

Manages contracts for the club's race services business, as well as for signature race sponsors. Replaces what used to be a manual Google Sheets workflow.

- Admin site: https://contracts.loutilities.com/
- Repo: https://github.com/louking/contracts
- Docs: https://docs.contracts.loutilities.com/en/latest/
- Repo's own `CLAUDE.md` (authoritative, deeper detail than this page): https://github.com/louking/contracts/blob/master/CLAUDE.md
- Uses the Google Docs API to generate and store contracts, and the Google Sheets API for related tracking
- Provides public views for race calendar availability
- Uses d3js to draw internal tracking charts

## Tech stack

- Backend: Python 3.12, Flask 3.0, SQLAlchemy 1.4, Alembic (migrations)
- Database: MySQL 8.0 (via PyMySQL driver — see gotcha below)
- Frontend: Jinja2 templates, WTForms, Flask-Assets/webassets
- Auth: Flask-Security-Too, Flask-Principal
- Document generation: python-docx, html2docx
- External integrations: RunSignup (race data), Google Workspace/Sheets API
- Mail: Flask-Mail + msmtp
- Infrastructure: Docker, Docker Compose, Nginx, Gunicorn, phpMyAdmin, crond
- Shared libraries: [loutilities](../framework/loutilities.md)

## Architecture notes

- **Blueprints**: `views/admin/` (admin route handlers), `views/frontend/` (public race calendar availability), `views/userrole/` (user/role management via loutilities).
- **Contract generation**: `contractmanager.py` holds the core logic, producing contracts via the Google Docs API from templates.
- **Config**: `config/contracts.cfg` (email addresses, API keys, timing thresholds) and `config/users.cfg` (user auth), with DB passwords mounted as Docker secrets under `config/db/`.
- **Testing**: has a `pytest` suite under `test/` — notably more test coverage than most of the other xtility apps.
- **Deployment**: Fabric (`fabfile.py`), pulling and restarting the Docker stack on the target host. Production runs behind **Caddy** as the HTTPS reverse proxy (shared Caddyfile covering all loutilities apps — see [infrastructure/caddy.md](../infrastructure/caddy.md)).

## Gotchas worth knowing

- **JS assets are shadowed by a Docker volume mount** to the shared `js-common` directory, same pattern as membertility/scoretility/routetility — editing `static/js/` in the repo has no effect on the running container.
- **MySQL 8 + Alpine containers**: `mysqlclient` fails TLS certificate verification against MySQL 8's auto-generated self-signed certs. Fix is switching to the pure-Python **PyMySQL** driver (`mysql+pymysql://`) — same fix applied across the other xtility apps.
- **Caddy HTTP/3/QUIC issue**: Caddy's default HTTP/3 (QUIC) can cause intermittent `net::ERR_QUIC_PROTOCOL_ERROR` in Chrome (page returns 200 but renders blank) on larger responses. Fixed by adding `protocols h1 h2` to the Caddyfile's global `servers` block to disable HTTP/3 — a Caddy transport issue, not an app bug. See [infrastructure/caddy.md](../infrastructure/caddy.md).

For full detail on any of the above (project structure, key entry points, full config reference), see the repo's own `CLAUDE.md` linked above.
