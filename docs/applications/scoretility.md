# scoretility

Manages scoring for the steeplechasers series (grand prix, equalizer, etc.), as well as the Maryland/DC Grand Prix. Collects data and processes results to provide end-of-year analysis for major running awards.

- Public site: [scoretility.com](http://scoretility.com/)
- Repo: https://github.com/louking/rrwebapp (this is the oldest of the apps)
- Docs: https://docs.scoretility.com/en/latest/
- Repo's own `CLAUDE.md` (authoritative, deeper detail than this page): https://github.com/louking/rrwebapp/blob/master/CLAUDE.md

## Multi-club, but not the "interest" model

scoretility is multi-club enabled — it supports separate membership, races, divisions, series, results, and standings per club — but predates the [interest](../GLOSSARY.md) model used elsewhere in the framework. Instead of an `interest` URL segment and `g`-based lookup, club scoping is done with plain Flask session state: `session['club_id']` and `session['year']`, checked pervasively throughout views and models (`getclubid`/`getyear` lambdas in `model.py` are convenience accessors for forms). Every query filters by club ID, and access control (`accesscontrol.py`) uses Flask-Principal permissions (`UpdateClubDataPermission`, `ViewClubDataPermission`) scoped the same way.

## Tech stack

- Backend: Python 3.12, Flask 3.0.3, SQLAlchemy 2.x
- Database: MySQL 8.0.40 via PyMySQL driver (SQLite in-memory for testing) — see gotcha below
- Task queue: Celery 5.4 with RabbitMQ, for async results processing
- Frontend: server-rendered Jinja2 with DataTables, Flask-Assets for JS/CSS bundling
- Auth: Flask-Security-Too with Flask-Principal for role-based access control
- Shared libraries: [loutilities](../framework/loutilities.md) (`DbCrudApi` CRUD base, age-grade calculations, `timeu` utilities), `runtilities` (race result parsing)

## Architecture notes

- **Blueprints**: `admin` (`/admin` prefix) — club, member, race, results, resultsanalysis, standings, agegrade, location, services, uploads, userrole, debug; `frontend` (no prefix) — index, userviews, sysinfo.
- **External results import**: race results can be pulled from multiple sources, each with its own module — local file parsing (Excel/CSV/TXT), Athlinks API, Ultrasignup, RunningAHEAD.
- **CRUD views**: most admin views extend `loutilities`' `DbCrudApi`, which handles DataTables server-side processing, Editor integration, and CRUD/JSON endpoints.
- **Celery**: two queues — the default `celery` service for regular tasks, and a `longtask` queue (`celerylongtask` service) for long-running results imports; both run at concurrency 1.
- **Age grade models**: `AgeGradeTable` → `AgeGradeCategory` → `AgeGradeFactor` in `model.py`. Distance is stored as `dist_mm = int(dist_km * 1_000_000)` (millimetres); `AgeGradeCategory.oc_secs` holds the open-class (world record) performance in seconds for that distance.
- **External API credentials are database-backed, not config-backed** — unlike contractility/membertility (single-org, credentials read once from Flask config), scoretility is multi-club, so `ApiCredentials` (one row per service, keyed by name) and `RaceResultService` (links a club to a service + per-club JSON attrs) live in the DB and are admin-editable. Same `rsu_api_reg` token requirement as contractility (see its RunSignUp gotcha), stored as `ApiCredentials.api_reg_token`/`api_reg_secret`.
- **Testing**: first-ever `pytest` suite added 2026-08-07, following contractility's `pytest.ini`/`conftest.py` pattern (`APP_NAME` env default before package import, since `rrwebapp/__init__.py` reads it at module level). Also needed `APP_VER` (same import-time-`environ` pattern, in `version.py`) and — unlike contractility, where this was flagged but left unfixed — actually added the missing `Testing.EXCEPTION_EMAIL`/`APP_LOUTILITY`/`SECURITY_EMAIL_SUBJECT_*` config keys that `loutilities.user.applogging.setlogging()` and `create_app()` need unconditionally.

## Gotchas worth knowing

- **JS assets are shadowed by a Docker volume mount** to an external `js-common` directory (shared across apps, same pattern as membertility) — editing `app/src/rrwebapp/static/js/` in the repo directly has no effect on the running container.
- **Config** is read from `/config/rrwebapp.cfg`, with secrets (DB password, RabbitMQ password) mounted as Docker secrets. The `app` container's crontab is mounted the same way now, from `./config/cronjobs` — a runtime bind mount rather than baked into the image at build time, so the cron schedule can change without a rebuild.
- **Custom DataTables buttons** follow a two-file JS pattern: `beforedatatables.js` defines button handler functions that must exist before DataTables initializes (a button's Python `action` key is a string `eval()`'d client-side), and `afterdatatables.js` holds per-path post-init hooks.
- **`RaceResult`'s unique constraint doesn't actually prevent duplicates**: it includes `runnername`, which the tabulate flow never sets (always `NULL`), and SQL unique constraints treat `NULL` as distinct from `NULL`. A fast double-click on a button using the shared `ajax_update_db_noform()` JS helper (which doesn't disable itself) can fire two overlapping tabulate requests that both pass the "results already exist" check before either commits, doubling every result. Mitigated client-side for the Tabulate button by disabling on click and re-enabling via a document-level `ajaxComplete` listener; other buttons using that helper remain exposed.
- **Don't `pip install passlib` directly** — same gotcha as contractility: Flask-Security-Too's declared dependency is `passlib`, but real (unmaintained) `passlib` breaks under `setuptools>=83` (dropped `pkg_resources`; `passlib/pwd.py` does an unconditional `import pkg_resources`). Fix is `libpass`, an actively-maintained fork that installs into the same `passlib/` import path — `requirements.txt` has `libpass==1.9.3`, no separate `passlib` entry. Installing real `passlib` on top silently overwrites `libpass`'s files and reintroduces the break.
- **MySQL 8 + Alpine containers**: `mysqlclient` triggers a TLS/SSL certificate verification failure (Alpine's MariaDB Connector/C defaults to SSL). Fix is to switch to the pure-Python **PyMySQL** driver (`mysql+pymysql://` URI scheme) instead of patching SSL settings — same fix and same gotcha as membertility. `apk add mysql-client` (the CLI, for `mariadb`/`mariadb-dump` cron backups) is unrelated and stays either way. There are two separate connection strings needing this scheme, not one: the main app DB URI (`settings.py`) and Celery's result-backend URI (`celery.py`, `celeryconfig['result_backend']`). Missing the second isn't just a status-view cosmetic issue — the results-import task itself calls `self.update_state(...)` periodically from inside its own outer `try/except`, and when that call hits the missing driver, the bare `except:` catches it, rolls back the DB session (discarding every result row accumulated so far), and reports fake "success" with the traceback embedded. So the bug can silently roll back entire imports, not just fail to report their progress.
- **`runtilities` needs `>=3.0.1`, not just `>=3.0.0`**: `runtilities==3.0.0`'s `running.runningahead` module had a module-level `import version` with no top-level `version` package to resolve it — always fails. Harmless for contractility/membertility (they never import `running.runningahead`, only `running.runsignup`), but fatal for scoretility, which imports it for its RunningAHEAD results-analysis integration and hit it on every app startup. Fixed upstream in the `running` repo (the dead code was leftover from a non-functional `main()` CLI stub — `argparse`'s `version=` kwarg was removed from Python 3 years ago, so that function had never actually run).

For full detail on any of the above (directory layout, all CLI/deployment commands via Fabric, results-analysis debug flags), see the repo's own `CLAUDE.md` linked above — treat it as the authoritative, actively-maintained source rather than duplicating it here.
