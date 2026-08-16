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
- **Google Maps loads async, gated by an inline `onGmapsReady` stub**: the Maps script tag isn't in an asset bundle (bundled `<script>` tags can't carry `async`); each map-using template loads it directly with `loading=async&callback=gmapsCallback`, and the corresponding page JS defers its map-dependent init until both the DOM and that callback have fired. The `gmapsCallback`/`onGmapsReady` registration must be a tiny inline `<script>` ahead of the async tag, not defined inside the page's own JS bundle — that bundle loads behind the ~2MB shared common bundle and loses the race against the (much smaller) async Maps script in practice. Extending `google.maps.OverlayView` for the custom SVG overlay must use `Object.setPrototypeOf`, not prototype replacement, since that setup happens after parse time. On the route detail page, Maps' own `onAdd()` callback and the route-data AJAX response can complete in either order, so the elevation chart's distance array must be computed unconditionally as soon as route data arrives, not as a side effect of `onAdd()` having fired.
- **Bundle URLs already get cache-busting query strings by default**: `webassets`' own defaults (hash-based `versions`, no explicit `url_expire`) append a `?<hash>` to every generated bundle URL without any extra `Environment` config — a rebuilt bundle gets a new URL automatically. (Don't assume a bare `Environment()` means no cache-busting; check the installed `webassets` version's defaults first.)
- **Each JS bundle in `assets.py` needs a unique `output=` path**: two bundles sharing one output path silently overwrite each other's compiled file on rebuild, serving one page's JS to another.
- **`pytest` suite** covers `helpers.py`, `geo.py`'s `GmapsLoc` (mocked `googlemaps.Client`, no real network calls), `models.py`'s free functions/`update_local_tables()`, `files.py`, `locations.py`, `dbinit.py`, `nav.py`'s role-based menu construction, and — on the `DbCrudApiRolePermissions`-based admin views and the frontend `MethodView`s — the override methods that don't themselves call `render_template()` (`permission()`, `beforequery()`, `snaploc()`, etc.), called directly on the module-level view singletons rather than through the full DataTables request/response protocol. Several modules (`locations.py`, `nav.py`, the admin/frontend route/icon views) build a real `googlemaps.Client`/register with `flask_nav` at *module import time* using the app-level config, so they're only importable once `create_app()` has run once in the process — tests defer those imports via a fixture rather than a top-level `import`. `create_app()` also rebinds Flask-SQLAlchemy's `db.session` proxy to whichever app called it last, so a test session mixing the full `app`/`dbapp` fixtures with the lightweight `bareapp`/`bare_dbapp` ones needs a `_rebind_session()` helper called between fixture types to avoid order-dependent failures.
- **`libpass` must be installed *alongside* `passlib`, not instead of it**: `libpass` 1.9.3 only ships a partial `passlib`-namespace shim (no `context.py`/`CryptContext`), so removing `passlib` entirely breaks `flask_security.core`'s import. Both packages need to be present together, matching membertility's combination.

For full detail (directory layout, deployment via Fabric, config files), see the repo's own `CLAUDE.md` linked above.
