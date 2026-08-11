# contractility

Manages contracts for the club's race services business, as well as for signature race sponsors. Replaces what used to be a manual Google Sheets workflow. For the technical/operational side of race timing (chip timing, backup timing, finish-line video) — used here and for the club's own low-key races — see [race-services/](../race-services/).

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
- **Testing**: has a `pytest` suite under `test/` with over 100 tests — notably more test coverage than most of the other xtility apps, following the same test-expansion pass done for scoretility. Run `pytest` from the repo root; `pytest.ini` puts `app/src` on `sys.path`, and `test/conftest.py` sets `APP_NAME` (normally supplied by Docker Compose's `.env`) since `contracts/__init__.py` reads it at import time. Two fixture pairs exist: `app`/`dbapp` (goes through the real `create_app()`, for tests needing routing/security) and `bare_dbapp`/`bareapp` (a bare Flask app with just the db bound, for model/free-function-level tests) — same split as scoretility's `dbapp`, though the reason differs (there it's celery's container-only config reads; here it's a `create_app()` ordering bug, see gotcha below).
- **Deployment**: Fabric (`fabfile.py`), pulling and restarting the Docker stack on the target host. Production runs behind **Caddy** as the HTTPS reverse proxy (shared Caddyfile covering all loutilities apps — see [infrastructure/caddy.md](../infrastructure/caddy.md)).

## Gotchas worth knowing

- **JS assets are shadowed by a Docker volume mount** to the shared `js-common` directory, same pattern as membertility/scoretility/routetility — editing `static/js/` in the repo has no effect on the running container.
- **MySQL 8 + Alpine containers**: `mysqlclient` fails TLS certificate verification against MySQL 8's auto-generated self-signed certs. Fix is switching to the pure-Python **PyMySQL** driver (`mysql+pymysql://`) — same fix applied across the other xtility apps.
- **Caddy HTTP/3/QUIC issue**: Caddy's default HTTP/3 (QUIC) can cause intermittent `net::ERR_QUIC_PROTOCOL_ERROR` in Chrome (page returns 200 but renders blank) on larger responses. Fixed by adding `protocols h1 h2` to the Caddyfile's global `servers` block to disable HTTP/3 — a Caddy transport issue, not an app bug. See [infrastructure/caddy.md](../infrastructure/caddy.md).
- **RunSignUp client subclasses `running.runsignup.RunSignupBase`** (from the `runtilities` PyPI package) instead of vendoring the full client, adding only contractility-specific coupon-management and participant-list methods. Watch for a transitive-dependency gotcha: importing `running.runsignup` pulls in `loutilities.csvwt`, which unconditionally requires `openpyxl` — added to `requirements.txt` solely to satisfy that import chain. Endpoints target `api.runsignup.com`, and the client sends the `rsu_api_reg` token RunSignUp will require on every call starting 2027-01-01.
- **`test/conftest.py` had a stale package-rename bug**: it imported from `racesupportcontracts` (the app's pre-rename name) instead of `contracts`, at module level — this failed at conftest *collection* time, silently breaking every test in `test/`, not just the ones using the `app`/`dbapp` fixtures. Fixed alongside adding `pytest.ini` (`app/src` on `sys.path`) and an `APP_NAME` env default in conftest (needed since `contracts/__init__.py` reads it at import time, normally supplied by Docker Compose).
- **`test_basic.py::test_login` still errors, but not for the reason once suspected**: `settings.Testing` was missing `EXCEPTION_EMAIL`/`APP_LOUTILITY`/the `SECURITY_EMAIL_SUBJECT_*` keys and a `SQLALCHEMY_BINDS['users']` entry (all now filled in) — but even with those fixed, `create_app()` unconditionally queries the `Application` table for `g.loutility` *while creating the app*, before the `dbapp` fixture has run `db.create_all()`. That's a real ordering bug in `create_app()`/`conftest.py`, left unfixed since correcting it means changing production startup code, not test config.
- **Don't `pip install passlib` directly** — Flask-Security-Too here depends on `libpass` (an actively-maintained passlib fork), which installs its files *into* the `passlib/` import path so `flask_security`'s `from passlib... import ...` code works unmodified. Installing the real, unmaintained `passlib` package on top silently overwrites those files, which then breaks on `passlib.pwd`'s unconditional `import pkg_resources` (modern `setuptools` no longer ships it). Symptom looks like a missing/broken dependency; the fix is `pip uninstall passlib && pip install --force-reinstall --no-deps libpass==<pinned version>`, not adding `passlib` anywhere.

For full detail on any of the above (project structure, key entry points, full config reference), see the repo's own `CLAUDE.md` linked above.
