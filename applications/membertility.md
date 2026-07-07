# membertility

Club management app with several modules.

- Admin site: https://members.loutilities.com/admin
- Repo: https://github.com/louking/members
- Docs: https://docs.members.loutilities.com/en/latest/
- Repo's own `CLAUDE.md` (authoritative, deeper detail than this page): https://github.com/louking/members/blob/master/CLAUDE.md

## Modules

- **organization** — maintains the club leadership organization, committees, etc.
- **tasks** — for people in certain positions, manages tasks they need to do, mostly for onboarding (e.g., read document x, fill out this form, take this online course); manages racing team contracts; tracks coach training
- **meetings** — manages club meetings (board meetings, etc.), maintaining RSVPs, status reports, agenda, minutes, action items, motions, and votes (e-voting)
- **membership** — caches club membership information as collected from the registration service (RunSignup); provides public views into membership status and statistics; uses d3js to draw membership statistics
- **racing team** — provides user forms to (1) collect applications, and (2) collect information about races run and volunteerism; provides a database for admins to track racing team members over time
- **awards** — tracks race awards and associated competitions (RunSignup-based)
- **community** — manages the FSRC Community forum (built on Discourse): group sync (from RunSignup races/clubs or internal position tags), event topic creation, per-series calendar (`.ics`) feeds, and forum taxonomy export
- **super admin** — system-level configuration: SSO/auth, email, file storage, user roles, interest attributes

## Tech stack

- Backend: Python 3.12, Flask 3.0, SQLAlchemy 2.0, Flask-Security-Too, Flask-Migrate (Alembic)
- Database: MySQL 8.0 (via PyMySQL driver — see gotcha below)
- Frontend: Jinja2 templates, Webassets (CSS/JS bundling), Bootstrap
- Infrastructure: Docker + Docker Compose, Nginx reverse proxy, Gunicorn
- External integrations: Google Workspace API, MailChimp, RunSignup API, Discourse API, Mailgun (msmtp)
- Shared libraries: [loutilities](../framework/overview.md) (custom framework), `runtilities`

## Architecture notes

- **Multi-tenancy**: URL structure is `/app/<interest>/admin/...`. The `interest` (e.g., which club/series) is stashed in Flask's `g` via a URL preprocessor; views use a `localinterest()` helper to read it.
- **Blueprints**: admin-facing views (`views/admin/`) are separate from member-facing views (`views/frontend/`).
- **Config**: INI-format files under `config/`, mounted read-only in Docker; secrets come from Docker secrets (`/run/secrets/`). Values are run through `eval()` by `loutilities.configparser`, so booleans/ints/dicts/lists arrive as native Python types, not strings.
- **CLI**: four Click command groups run as `flask <group> <command>` — `members`, `membership` (e.g., MailChimp sync), `task`, `community` (FSRC Community group sync, calendar feed generation, event import, taxonomy export).
- **Calendar feeds**: served on-demand at `/<interest>/calendars/<series>.ics` by the Flask app itself (not static files, not cron) — in-memory caching (1 hr for tag lookups, 15 min for compiled ICS bytes).

## Gotchas worth knowing

- **JS assets are shadowed by a Docker volume mount** to an external `js-common` directory (shared across apps). Editing `app/src/members/static/js/` in the repo directly has no effect on the running container.
- **MySQL 8 + Alpine containers**: `mysqlclient` triggers a TLS/SSL certificate verification failure (Alpine's MariaDB Connector/C defaults to SSL). Fix is to switch to the pure-Python **PyMySQL** driver (`mysql+pymysql://` URI scheme) instead of patching SSL settings.
- **Discourse API quirks**: booleans must be passed as lowercase strings (`'true'`/`'false'`, not Python `True`), several endpoints return unexpectedly shaped data (e.g., `site_settings.json` returns a list; topic `tags` may be strings or objects), and category group permissions aren't exposed by any Discourse REST endpoint — they're fetched via a Data Explorer SQL query instead.
- **No test suite** in the main app itself; `loutilities` has its own tests (`python -m pytest tests/`).

For full detail on any of the above (directory layout, all CLI commands, deployment via Fabric, Discourse client patterns), see the repo's own `CLAUDE.md` linked above — treat it as the authoritative, actively-maintained source rather than duplicating it here.
