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
- Database: MySQL 8.0.40 (SQLite in-memory for testing)
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

## Gotchas worth knowing

- **JS assets are shadowed by a Docker volume mount** to an external `js-common` directory (shared across apps, same pattern as membertility) — editing `app/src/rrwebapp/static/js/` in the repo directly has no effect on the running container.
- **Config** is read from `/config/rrwebapp.cfg`, with secrets (DB password, RabbitMQ password) mounted as Docker secrets.
- **Custom DataTables buttons** follow a two-file JS pattern: `beforedatatables.js` defines button handler functions that must exist before DataTables initializes (a button's Python `action` key is a string `eval()`'d client-side), and `afterdatatables.js` holds per-path post-init hooks.

For full detail on any of the above (directory layout, all CLI/deployment commands via Fabric, results-analysis debug flags), see the repo's own `CLAUDE.md` linked above — treat it as the authoritative, actively-maintained source rather than duplicating it here.
