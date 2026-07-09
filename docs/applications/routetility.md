# routetility

A database of running routes.

- Public site: [routes.loutilities.com](http://routes.loutilities.com/)
- Repo: https://github.com/louking/runningroutes
- Docs: https://docs.routes.loutilities.com/en/latest/
- Repo's own `CLAUDE.md` (authoritative, deeper detail than this page): https://github.com/louking/runningroutes/blob/master/CLAUDE.md
- Uses the Google Maps API to draw a map with all routes' start points and each route's path, and to fetch elevation/geocoding data
- Uses d3js to draw an elevation chart

## Tech stack

- Backend: Flask, SQLAlchemy, Flask-Migrate (Alembic)
- Database: MySQL 8.x, with two binds — the default DB (routes/icons/locations) and a `users` bind (Flask-Security-Too auth tables, shared with other apps via loutilities)
- Auth: Flask-Security-Too
- Shared libraries: [loutilities](../framework/loutilities.md) (`DbCrudApiRolePermissions` CRUD base, shared user/role models, geolocation utilities)

## Architecture notes

- **Multi-tenant via [interest](../GLOSSARY.md)**: like membertility (and unlike the older [scoretility](scoretility.md)), routetility uses the `interest` model — every admin view URL starts with `/<interest>/`, and each blueprint's `__init__.py` registers a `pull_interest()` preprocessor that reads the URL segment into `g.interest`.
- **Blueprints**: `admin` (`/admin` prefix) — routes (CRUD + elevation fetch), icons, icon-locations, file upload, user/role admin, sysinfo/debug; `frontend` (no prefix) — public route listing/search, public icon map.
- **`LocalInterest`/`LocalUser` sync**: local copies of the loutilities central auth tables, synced via `update_local_tables()` on startup — allows interest-scoped queries without a cross-database join.
- **CRUD views**: subclass `DbCrudApiRolePermissions` from loutilities; `beforequery()` applies the interest-scoped filter (`self.queryparams['interest_id'] = self.linterest.id`).
- **Roles**: `ROLE_SUPER_ADMIN` (bypasses interest-scoped checks), `ROLE_ROUTES_ADMIN`, `ROLE_ICON_ADMIN`.
- **Elevation**: `GmapsLoc` (in `geo.py`) wraps the Google Maps API, sampling path points (≤512 samples, ~60 ft apart) to compute elevation gain.
- **File uploads**: GPX/KML uploads and processed paths are tracked by the `Files` model and stored under a Docker volume.

## Gotchas worth knowing

- **JS assets** are shadowed by a `JS_COMMON_HOST` Docker volume mount (same shared `js-common` directory pattern as membertility/scoretility) — editing the repo's own `static/js/` has no effect on the running container.
- **d3-tip patched for D3 v7**: the live file lives in `js-common`, patched from the VACLab fork to guard `d3.event.target` (removed in D3 v7) — there's no maintained upstream D3-v7-compatible version, so don't replace it wholesale.
- **`rjsmin` required for JS bundling**: `assets.py` filters JS bundles with `rjsmin` (not `jsmin`); if missing, production bundle rebuilds fail and debug mode silently skips minification.
- **`mutex-promise.js` bundle ordering**: must load before loutilities' `datatables.js` in both JS bundles, or `MutexPromise` is undefined and DataTables editor buttons fail to register.
- **No automated test suite**; `/admin/test-exception` exercises error-handling in a running deployment.

For full detail (directory layout, deployment via Fabric, config files), see the repo's own `CLAUDE.md` linked above.
