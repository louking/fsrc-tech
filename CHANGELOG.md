# Changelog

A running log of notable updates to this wiki. Newest entries at the top.

## 2026-08-11

- `applications/contractility.md`: expanded the `pytest` suite from 18 tests to over 100, following the same test-expansion pass done for scoretility on 2026-08-09 — new coverage for `daterule.py`, `apicommon.py`, `html2docx.py`, `dbmodel.py`'s `DateRule`/`ModelItem`/`getmodelitems`/`initdbmodels`, `utils.py`'s `renew_event`/`renew_sponsorship`, `trends.py`'s `calculateTrend`/sponsorship-conflict checks, and `contractmanager.py`'s template/merge-field logic. Added a `bare_dbapp`/`bareapp` fixture pair (bare Flask app with just the db bound, no `create_app()`) for the model-level tests, mirroring scoretility's `dbapp` pattern — though for a different reason: not celery's container-only config reads, but a real ordering bug found in `create_app()` itself, which queries the `Application` table for `g.loutility` before `db.create_all()` has run. That means `test_basic.py::test_login`'s pre-existing failure (flagged since 2026-08-06) was misdiagnosed — filling in the missing `Testing.EXCEPTION_EMAIL`/`APP_LOUTILITY`/`SECURITY_EMAIL_SUBJECT_*`/`SQLALCHEMY_BINDS['users']` config keys (now done) gets past the original error but doesn't fix it; the real cause is that ordering bug, left as a known gap since fixing it means changing `create_app()` itself. Also surfaced two smaller findings along the way: `request.py`'s `annotatescripts`/`addscripts`/`crossdomain` have no callers anywhere in the app (confirmed dead code); and `contractmanager._evaluate()`'s callable-leaf detection only actually invokes callables lacking a `__dict__` (e.g. `__slots__`-based), so ordinary functions/lambdas assigned as merge-field values silently pass through uncalled — harmless in practice since production code only ever passes SQLAlchemy model instances as merge fields. Pulled from `contracts`'s own `CLAUDE.md` per this wiki's sync convention.

## 2026-08-09

- `applications/scoretility.md`: expanded the `pytest` suite from 5 tests to 81, covering every top-level module that's importable under the bare-`Flask` test fixture (`model.py`, `resultsutils.py`, `clubmember.py`, `helpers.py`, `raceresults.py`, `athlinksresults.py`, `ultrasignupresults.py`, `runningaheadresults.py`). Confirmed by actually trying the import, not just inspection, that `tools.py` — despite sitting at the top level rather than under `views/admin/` — still pulls in the same `celery.py` config-read chain (via `views.admin.member`) and so isn't testable this way; being outside `views/admin/` turned out not to be sufficient on its own. Writing the coverage surfaced two pre-existing bugs, left unfixed as out of scope for a test-writing pass: `resultsutils.ServiceAttributes.set_attr()` queries `RaceResultService` by a `name` kwarg the model doesn't have (and is never actually called anywhere); and `athlinksresults.AthlinksCollect.getresults()`'s age-group "fuzzy match" branch used `(x/5)*5`-style checks written for Python 2's truncating division, which under Python 3's true division silently never matches anything — fuzzy-age matching hasn't worked since the Python 3 port. Pulled from `rrwebapp`'s own `CLAUDE.md` per this wiki's sync convention.
- `applications/scoretility.md`: extended the MySQL 8 + Alpine PyMySQL gotcha — the `mysql+pymysql://` scheme has to be set on two separate connection strings, not one. The main app DB URI (`settings.py`) was fixed already, but Celery's result-backend URI (`celery.py`) was still plain `db+mysql://`. Initially thought this only broke the status-polling view (`ModuleNotFoundError: No module named 'MySQLdb'` on `/admin/importresultsstatus/<task_id>`), but a second traceback from the same incident showed it's worse: the results-import task itself calls `self.update_state(...)` inside its own outer `try/except`, so the same error gets caught there and triggers `db.session.rollback()` — silently discarding an entire in-progress import and reporting fake "success". Filed as [rrwebapp#685](https://github.com/louking/rrwebapp/issues/685) and fixed. Pulled from `rrwebapp`'s own `CLAUDE.md` per this wiki's sync convention.

## 2026-08-08

- `infrastructure/caddy.md`: noted that `fsrc-tech.localhost` in the dev Caddyfile is different from the other local `*.localhost` domains — it's served as static files directly (`file_server`, no proxy), with this wiki's own working directory bind-mounted into the caddy container via `FSRC_TECH_WWW_HOST`. Pulled from `caddy-docker`'s own `CLAUDE.md` per this wiki's sync convention.
- `applications/scoretility.md`: added the passlib/libpass gotcha — Flask-Security-Too's declared dependency is `passlib`, but real (unmaintained) `passlib` breaks under `setuptools>=83` (`passlib/pwd.py`'s unconditional `import pkg_resources`); fix is `libpass`, an actively-maintained fork installing into the same `passlib/` import path. Same fix already documented for contractility on 2026-08-06; scoretility hit the identical `ModuleNotFoundError: No module named 'pkg_resources'` trace (via `flask_security.core` → `.totp` → `passlib.pwd`) when starting the app after a dependency bump, fixed by swapping `passlib==1.7.4` for `libpass==1.9.3` in `requirements.txt` and reinstalling locally. Pulled from `rrwebapp`'s own `CLAUDE.md` per this wiki's sync convention.

- `applications/scoretility.md`: noted that the `app` container's crontab (`/etc/crontabs/appuser`) is now mounted at runtime from `./config/cronjobs`, same pattern as `rrwebapp.cfg`, rather than `COPY`'d into the image at build time — so the cron schedule can be changed without an image rebuild. Pulled from `rrwebapp`'s own `CLAUDE.md` per this wiki's sync convention.
- `applications/scoretility.md`: added the MySQL 8 + Alpine SSL gotcha and its fix — switched from `mysqlclient` to the pure-Python PyMySQL driver (`mysql+pymysql://` URI scheme), same fix membertility already documented. Caught mid-refactor: an unstaged `requirements.txt` change had dropped `mysqlclient` with no replacement driver while `settings.py` still built a plain `mysql://` URI, which would have failed to connect in production; fixed by adding `PyMySQL==1.1.3` and updating the URI scheme. Pulled from `rrwebapp`'s own `CLAUDE.md` per this wiki's sync convention.

## 2026-08-07

- `applications/scoretility.md`: added scoretility's first-ever `pytest` suite (following contractility's `pytest.ini`/`conftest.py` pattern — `APP_NAME` env default before package import), plus an `APP_VER` default and the `Testing.EXCEPTION_EMAIL`/`APP_LOUTILITY`/`SECURITY_EMAIL_SUBJECT_*` config keys `loutilities.user.applogging.setlogging()`/`create_app()` need unconditionally (contractility flagged the same `EXCEPTION_EMAIL` gap in its own `test/` suite on 2026-08-06 but left it unfixed; scoretility's pass fixed it). Documented the database-backed `ApiCredentials`/`RaceResultService` credential pattern (vs. contractility/membertility's Flask-config-based credentials) used to add RunSignUp's `rsu_api_reg` token support ahead of its 2027-01-01 deadline. Also caught and fixed a real bug in the shared `runtilities` package itself: `runtilities==3.0.0`'s `running.runningahead` had a module-level `import version` that always failed, breaking scoretility's app startup (it imports that module; contractility/membertility don't) — fixed upstream, now requires `runtilities>=3.0.1`. Pulled from `rrwebapp`'s own `CLAUDE.md` per this wiki's sync convention.

## 2026-08-06

- `applications/contractility.md`: added pytest coverage for the RunSignUp refactor (`test/test_runsignup.py`, `test/test_helpers.py`) and fixed what turned out to be a fully broken `test/` suite — `conftest.py` imported from the app's pre-rename package name (`racesupportcontracts` instead of `contracts`) at module level, which failed at conftest collection time and silently broke every test in the directory, not just the ones using the `app`/`dbapp` fixtures. Added `pytest.ini` (`app/src` on `sys.path`) and an `APP_NAME` env default in conftest, since `contracts/__init__.py` reads it at import time and it's normally only supplied by Docker Compose's `.env`. Also documented a nasty local-venv trap hit along the way: installing real `passlib` on top of `libpass` (an actively-maintained fork Flask-Security-Too here actually depends on, which installs its files into the `passlib/` import path) silently overwrites it and reintroduces passlib's `pkg_resources` transitive-import break — the fix is reinstalling `libpass`, not adding `passlib`. One pre-existing test (`test_basic.py::test_login`) still errors for an unrelated reason (`Testing.EXCEPTION_EMAIL` missing from `settings.py`) — flagged but not fixed, out of scope for this pass. Pulled from `contracts`'s own `CLAUDE.md` per this wiki's sync convention.

## 2026-08-05

- `race-services/timing.md`: corrected the LS/BS explanation in "Last Seen, First Seen, and Best Seen" — RDS doesn't select LS vs. BS as separate feeds; it takes the last read received (by timestamp) at the start line and the first read received (by timestamp) at the finish, which happens to resolve to LS from UHF8's perspective at the start and BS at the finish. Previous wording implied RDS was BS-aware, which isn't accurate.
- `race-services/timing.md`: corrected the Best Seen settling time twice — first from a fixed "about half a second" to "a configurable tag timeout, 5 seconds in FSRC's configuration," then further corrected per Trident's [Tag Settings](https://www.manula.com/manuals/tridentrfid/timemachine/1/en/topic/tag-settings) docs. Ultimately trimmed the Tag Timeout mechanics out of the paragraph entirely as too much detail for this page — the key point is what RDS does with the reads (last read at start, first read at finish), the order UHF8 forwards them (LS then BS), and why: the runner's body blocks further reads once they've crossed the start line, so no extraneous late reads arrive there, while at the finish reads can arrive early.

## 2026-08-02

- `applications/contractility.md`: trimmed the RunSignUp `RunSignupBase` gotcha bullet, which had ballooned into a 6-sentence paragraph duplicating method/config names and historical narrative already covered by `contracts`' own `CLAUDE.md` and this wiki's `CHANGELOG.md`. Kept only the subclassing fact, the `openpyxl` transitive-dependency trap, and the 2027-01-01 API token deadline.

## 2026-08-01

- `applications/membertility.md`: added a gotcha — `setuptools>=83` dropped `pkg_resources`, breaking `FormEncode==2.0.1`'s unconditional import of it; fixed by bumping to `FormEncode==2.1.1` upstream rather than pinning `setuptools` back down. Pulled from `members`'s own `CLAUDE.md` per this wiki's sync convention.

## 2026-07-31

- `applications/contractility.md`: reworked the RunSignUp client gotcha now that `running`/`runtilities` published a `RunSignupBase` class (shared auth/session/`_rsuget`/`_rsugetcsv` plumbing) and split `RunSignupFluent` (needing `universalclient`/`rauth`) into its own module — the dependency-coupling objection from the 2026-07-30 entries no longer applies. `contracts.runsignup.RunSignUp` now subclasses `RunSignupBase` instead of vendoring the full client, keeping only its coupon-management/POST and participant-list methods. Also documented a transitive-dependency gotcha this surfaced: `running.runsignup` imports `loutilities.csvwt` at module level, which unconditionally needs `openpyxl` — contractility had to add it to `requirements.txt` purely to satisfy that import chain. Pulled from `contracts`'s own `CLAUDE.md` per this wiki's sync convention.

## 2026-07-30

- `applications/contractility.md`: added a gotcha explaining why `contracts`' `runsignup.py` is a standalone vendored fork rather than using the `running`/`runtilities` package — different RSU endpoints needed (coupons/participants vs. memberships/results), no POST support upstream. Also flagged known debt: still on legacy `runsignup.com` endpoints, hasn't adopted the `api.runsignup.com` migration or the 2027-01-01 API registration token requirement that `running`'s client already handles. Pulled from `contracts`'s own `CLAUDE.md` per this wiki's sync convention.
- `applications/contractility.md`: updated the above gotcha — the legacy email/password Login API path in contractility's RunSignUp client is confirmed dead code (both call sites use `key=`/`secret=` only, no `RSU_EMAIL`/`RSU_PASSWORD` in config), and noted that centralizing auth via subclassing `running`'s client is a bad idea since it'd drag in `universalclient`/`rauth` as new dependencies — a shared `loutilities` module would be the right home instead. Pulled from `contracts`'s own `CLAUDE.md` per this wiki's sync convention.
- `applications/contractility.md`: contracts' RunSignUp client was updated — dead legacy email/password Login API path removed, endpoints migrated to `api.runsignup.com`, `rsu_api_reg` token/secret support added for RunSignUp's 2027-01-01 API registration requirement, and client instantiation centralized into `helpers.make_runsignup_client()` (mirroring the pattern already used in `members`). Pulled from `contracts`'s own `CLAUDE.md` per this wiki's sync convention.

## 2026-07-22

- Added `/wiki-lint` (`.claude/skills/wiki-lint/`), the "Lint" operation from the wiki's guiding pattern (see `CLAUDE.md`'s new "Guiding pattern for wiki maintenance" section, referencing Karpathy's [LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)): a quick mechanical pass (broken internal links, orphaned pages, unused glossary terms, CHANGELOG hygiene) plus an optional `deep` pass (source-repo drift, external anchor rot, Google Doc staleness, contradictions).
- `infrastructure/hosting.md` and `framework/loutilities.md`: added cross-linked pointers to Lou's separately-maintained development-process doc set, [process.loutilities.com](https://process.loutilities.com/) — mostly server/account setup, database migration, Python package release, Docker, and HTTPS (hosting.md), but it also covers development-environment setup and `loutilities`-specific `tables.py`/Editor coding patterns (loutilities.md), so it's linked from both pages rather than just hosting. Caution: parts predate the Docker/Rocky Linux migration and may be stale.
- `framework/loutilities.md`: added a `tables.py`/`tables-assets` gotcha — a button object overriding a standard Editor action (e.g. a conditionally-disabled `create` button) needs the shared Editor instance attached explicitly, since `get_button_options()` only does that automatically for the plain string form. Root-caused and fixed in `loutilities` from a live `members` crash (`Cannot read properties of null (reading 'i18n')` at table init). Pulled from `loutilities`'s own `CLAUDE.md` per this wiki's sync convention.
- `framework/loutilities.md`: updated the above gotcha now that it's fixed upstream (loutilities [3.12.4](https://github.com/louking/loutilities/commit/d3e2355)) — `get_button_options()` now injects `editor:` on object-form standard-action overrides too, so the workaround is no longer needed.

## 2026-07-19

- `applications/membertility.md`: updated the pending-review notifier's gotchas with two fixes from first live use — a `GET /review.json` pagination trap, and the PM now showing a readable title/submitter/local-time instead of a bare id and UTC timestamp. Pulled from `members`'s own `CLAUDE.md` per this wiki's sync convention.

## 2026-07-18

- `applications/membertility.md`: documented the community module's new pending-review notifier (`flask community notify-pending-reviews`), which PMs a category's moderator group(s) about stale Discourse review-queue items — a gap since Discourse's own moderation badge doesn't notify anyone. Pulled from `members`'s own `CLAUDE.md` per this wiki's sync convention.

## 2026-07-15

- add Pie Run 2025 to `race-services/reports/`

## 2026-07-14

- `applications/membertility.md`: the community module's calendar feed was refactored from per-series URLs (`/<interest>/calendars/<series>.ics`, one file per tag configured in `CALENDAR_TAG_GROUPS_<INTEREST>`) to a single `/<interest>/calendars/events.ics` endpoint taking a `tags=` query param (raw Discourse tag slugs, union match) plus `year=`/`from=`/`to=` date-window params, with `from=today` and an open-ended (all-future) default when `to` is omitted. Old URLs were retired outright, not redirected — a deliberate choice since no existing calendar subscriptions depended on them. That in turn made the disk-writing `filter-calendar` CLI command (and the `CALENDAR_TAG_GROUPS_<INTEREST>` config key / `get_tag_groups()` helper backing it) dead code — confirmed nothing else in the repo (cron, Nginx) referenced it — so it was deleted too, not just left in place. Pulled into `members`'s own `CLAUDE.md` first, then summarized here per this wiki's sync convention.

## 2026-07-13

- Fix summary legend, add # participants to report
- Added legend to `race-services/reports/`
- Added `org-tech/`, a new top-level topic covering third-party tech supporting general club operations (Google Workspace for Nonprofits, MailChimp, Facebook and the in-progress migration to FSRC Community) — the first real instance of the broader "other third-party tech" expansion this wiki's `CLAUDE.md` flagged as agreed-but-not-yet-scoped back on 2026-07-07. Seeded from Lou's [Digital Access and Identity Policy](https://docs.google.com/document/d/1dxsT7itCPQ_iHNLq_rNajqF9Ry72GxwGo9QmAflob4U/preview) (approved 2026-04-13) and a June 2026 [member communications survey](https://docs.google.com/presentation/d/1d6UbXKCqowknwfKrcoBn2PuCpGTy01scSH2YXnBV2VY/preview) (196 responses) on Facebook vs. forum sentiment — both condensed rather than transcribed. Added `Communications Chair`/`Technology Chair` glossary entries and updated the `MailChimp`/`FSRC Community` entries to cross-link.
- Published the Lewis Run 10 Miler (2026-01-10) sanitized report to `race-services/reports/`. 

## 2026-07-11

- Published the Wild Trail 5K & 10K (2026-04-04) sanitized report to `race-services/reports/`. First combined-distance race published here (5K and 10K run together off one shared start/timing setup, scored as two separate RDS events with different Gap Factors) — a clean result, 87 participants, only 1 missed start and 1 finish miss (recovered via backup timing). Updated `reports/README.md`'s running summary.
- `applications/scoretility.md`: added a gotcha about `RaceResult`'s unique constraint not actually preventing duplicate results (it keys on `runnername`, which the tabulate flow never sets, so it's always `NULL` and never matches). Root-caused from a doubled-results bug ([rrwebapp#677](https://github.com/louking/rrwebapp/issues/677)): a double-click on the Tabulate button could fire two overlapping requests that both raced past the "results already exist" check. Fixed client-side in that repo by disabling the button on click and re-enabling via `ajaxComplete`; pulled into `rrwebapp`'s own `CLAUDE.md` first, then summarized here per this wiki's sync convention.
- Migrated the wiki's build tooling from `mkdocs` + `mkdocs-material` to [Zensical](https://zensical.org), the Material for MkDocs team's successor static site generator. Prompted by the team's [MkDocs 2.0 warning](https://squidfunk.github.io/mkdocs-material/blog/2026/02/18/mkdocs-2.0/): MkDocs 2.0 removes the plugin system and rewrites theming with no migration path, which will break Material for MkDocs, now in maintenance mode. Zensical reads this repo's `mkdocs.yml` natively — no config changes needed since the site uses no plugins beyond `extra_css`. `mkdocs`/`mkdocs-material`/etc. uninstalled from the local `.venv`, `zensical` installed in their place; `local-notes/mkdocs-operational-setup.md` updated to the new commands (`zensical build`/`zensical serve`) and package name.
- Settled on `fsrc-tech.steeplechasers.org` as the wiki's domain (was tentatively `wiki.steeplechasers.org`); updated `mkdocs.yml`'s `site_url`, `CLAUDE.md`, and `local-notes/mkdocs-operational-setup.md` accordingly.
- `docs/stylesheets/extra.css`: the race-timing summary table's Notes column now clamps to two lines (`-webkit-line-clamp`) instead of just being widened, so a long note can no longer inflate row height; also widened the column further (16em → 28em) since two lines at the old width was still cutting notes off too early.
- `local-notes/mkdocs-operational-setup.md`: added a production-parity local testing step using a local `caddy-docker` instance (separate `.env`/`docker-compose.yml` from production) serving `fsrc-tech.localhost` from the repo checkout's `site/` build output, plus a day-to-day workflow step to verify via it before pushing. Corrected its `caddy trust` note: that command only installs the CA root into the trust store of whatever OS runs it, and Caddy runs inside the container, so it has to be run against the container's mounted data volume and the resulting cert imported into Windows' trust store directly, not run as `caddy trust` against the container.
- Added `.github/workflows/deploy.yml` and `requirements.txt`, completing the server-side rollout tracked in `local-notes/mkdocs-operational-setup.md`'s one-time setup checklist (deploy user, Caddy volume/block, SSH deploy key, and now repo secrets and the Actions workflow are all in place): pushes to `main` now build with Zensical and `scp` the output to the server automatically.
- `race-services/timing.md`: condensed the "RaceDay Scoring: per-race setup checklist" section, which had grown into a near-verbatim field-by-field transcription of the source [Chip/Scoring RaceDay Scoring Checklist](https://docs.google.com/spreadsheets/d/1GcDuB508jKu_kdMXZ8VhzIaZk6LE2l4qIxwpstLUKIc/edit?usp=sharing) sheet, down to a short pointer at that sheet plus the handful of config values (Trident/Time Machine stream IPs, folder paths, distance-based time limits) worth having on the page without opening it. Confirmed the sheet is link-shareable to anyone with the URL, not access-restricted as assumed going in.

## 2026-07-10

- Published the Parkway Panda 5K (2026-05-25) sanitized report to `race-services/reports/`. Notable: 12 participants ran without a chip assigned (a batch of bibs went out without one attached) and were timed entirely via the Time Machine backup stream — not a chip-reading failure. Updated `reports/README.md`'s running summary.

## 2026-07-09

- Added `race-services/reports/` — sanitized per-race chip-timing reports and a running cross-race summary, published from [chip-timing-analysis](https://github.com/louking/chip-timing-analysis)'s `reports/` output (aggregate-only, no bibs/names/race identity). Published the first entry, the 2026-07-04 5K. Linked from `race-services/README.md`.
- Refined `race-services/reports/`: each per-race report page now sets a `title` front-matter so the wiki nav shows the race date (not a generic heading) once there are many of these; the summary index (`README.md`) is retitled "Timing Summary", lists most-recent race first, drops the rarely-used "No Chip" column in favor of folding it into a short Notes cell when it happens, and truncates Notes generally so mkdocs-material's table doesn't blow up vertically wrapping long text.
- Fixed the Indy 5000 report's headline "chips not read at finish" count (was 4, now 3) and its % missed figure (2.3% → 1.8%): a confirmed drop (started, did not finish) has no finish read by definition and isn't a chip-reading miss, so it's no longer counted against these metrics — reported instead as its own "confirmed drop" line.
- Added `docs/stylesheets/extra.css` (wired via `mkdocs.yml`'s `extra_css`) to widen the last column of tables, so the timing-summary's Notes column doesn't wrap long text into tall rows.

## 2026-07-08

- Updated `applications/tmtility.md` and `race-services/timing.md`: tmtility's results view now shows the race Start Time (auto-populated from the live Trident `GUNTIME` marker, as already documented) for on-site verification during a race.

- Deepened `race-services/timing.md` from Lou's [Chip/Scoring RaceDay Scoring Checklist](https://docs.google.com/spreadsheets/d/1GcDuB508jKu_kdMXZ8VhzIaZk6LE2l4qIxwpstLUKIc/edit?usp=sharing) (Google Sheet, read via the Google Drive connector): added a "Chip vs. scoring-only races" distinction (whether Trident is deployed for a given race or Time Machine is the sole finish stream), a full per-race RaceDay Scoring setup checklist (race import, Scored Events, Timing Locations, Streams, sync/participants, age groups/awards, data checks, reports, wrap-up), and a post-race Trident log archival step (WinSCP upload to the network archive, then delete locally).

## 2026-07-07

- Removed `race-services/timing.md`'s Troubleshooting section (out of date). Also fixed its References section and a couple of inline mentions to link the actual source URLs — the initial Google Doc fetch used the plain-text export, which silently strips hyperlinks.
- Fixed a stale tmtility docs anchor in `race-services/timing.md`: `tm-csv-connector`'s user's guide removed the "Scanner Connected"/"Scanner Not Connected" distinction, so the old `#tmtility-operation-scanner-connected` anchor no longer exists — repointed to the current `#tmtility-operation` section.

- Added `race-services/` — a new top-level section for the technical/operational side of FSRC race timing (chip timing, backup timing, finish-line video), covering third-party hardware/software FSRC didn't build but configures and depends on. Used both for outside races FSRC contracts to time (the counterpart to contractility's contract/business side) and the low-key/informal races the club offers its own members. Seeded `race-services/timing.md` from Lou's own doc [Chip Timing Configuration and Operation](https://docs.google.com/document/d/1utAcupx2zqPyF6dONEBtDsQsB651qulztP_W2txLqsc/edit?usp=sharing): Trident chip-timing theory of operation, RaceDay Scoring/Trident/tmtility configuration reference, race-day operational checklist, and finish-line video backup (Topodome camera + Blue Iris). Added `race-services/README.md`, linked from root `README.md` and `CLAUDE.md`. Added glossary entries for Trident, GUNTIME, and Race Services; cross-linked from `applications/tmtility.md` and `applications/contractility.md`.
- Reworded the opening line of `README.md` and `CLAUDE.md` to lead with FSRC rather than ownership, noting the software is licensed to the club for free use in perpetuity (Lou credited as builder/maintainer, not owner-first). Also added a "Voice" section to `CLAUDE.md`: technical/architecture content stays passive, with "Lou" reserved for framing-level statements (README/CLAUDE.md, CHANGELOG tooling notes).
- Updated `applications/tmtility.md` gotchas with the Trident reader connectivity fix from `tm-csv-connector` (auto-reconnect, TCP keepalive, status debounce, louder results-page alert for degraded connectivity).
- Deepened the remaining application pages: `applications/routetility.md` and `applications/contractility.md` (tech stack, architecture, gotchas from their repos' own `CLAUDE.md`), `applications/tmtility.md` (hardware/client architecture, simulation mode, release pipeline, from `tm-csv-connector`'s `CLAUDE.md`), `applications/logmon.md` (services, background threads, two-database layout, from `logmon`'s `CLAUDE.md`), and `applications/wordpress-docker.md` (stack, deployment, gotchas, built directly from `docker-compose.yml`/`fabfile.py` since that repo has no `CLAUDE.md`). All apps now have deepened pages.
- Deepened `applications/scoretility.md`: added tech stack, architecture notes (blueprints, external results import, CRUD views, Celery queues, age grade models), and gotchas, sourced from the `rrwebapp` repo's own `CLAUDE.md`. Notes that scoretility predates the [interest](GLOSSARY.md) multi-tenancy model — it scopes clubs via `session['club_id']`/`session['year']` instead.
- Updated `infrastructure/hosting.md`: Cursor (not VS Code) is now Lou's primary dev editor.
- Added `infrastructure/caddy.md`: covers Caddy, the dockerized edge reverse proxy (github.com/louking/caddy-docker) that runs on the server and for local development — stack (caddy + certbot for CDN cert renewal), dev vs. production Caddyfile, deployment via Fabric, and the HTTP/3-disabled gotcha. Linked from `infrastructure/hosting.md` and `infrastructure/README.md`.
- Added `framework/loutilities.md`: deepened writeup of `loutilities`, covering `tables.py` (DataTables/Editor CRUD-grid integration: `CrudApi`, `DbCrudApi`, `DataTablesEditor`, field-type helpers, etc.) and `user/` (shared accounts/roles/multi-app SSO: models, role catalog, admin views, file uploads). Sourced from the repo's own `CLAUDE.md` plus direct inspection of `tables.py` and `user/*.py`.
- Updated `framework/overview.md` and `framework/README.md` to point to the new page.
- Added `xtility` glossary entry explaining the "-tility" app-suite naming convention.

## 2026-07-06 (6)

- Adopted "FSRC Community"/"FSRC Community forum" as the product-facing name for the club forum, reserving "Discourse" for when the underlying framework/API is actually the topic (parallel to steeplechasers.org vs. WordPress). Updated `GLOSSARY.md` and `applications/membertility.md` accordingly; documented the convention in `CLAUDE.md`.

## 2026-07-06 (5)

- Fixed "RunSignUp" → "RunSignup" (correct product capitalization) across the wiki.
- Sorted `GLOSSARY.md` entries alphabetically; noted the convention in `CLAUDE.md`.

## 2026-07-06 (4)

- Added `GLOSSARY.md` covering FSRC/domain-specific terms (FSRC, interest, Grand Prix, Equalizer, RunSignup, RaceDay Scoring, Time Machine, Discourse, MailChimp). Linked from `README.md`.

## 2026-07-06 (3)

- Added `LICENSE` (CC BY-SA 4.0) for the written content, and a License section in `README.md`. Noted this doesn't cover the application source code in the separate app repos.

## 2026-07-06 (2)

- Deepened `applications/membertility.md`: added tech stack, architecture notes (multi-tenancy, blueprints, config, CLI), and known gotchas, sourced from the `members` repo's own `CLAUDE.md`. Linked that file as the authoritative source for further detail.

## 2026-07-06

- Initial content seeded from Lou's [FSRC Technology Software Overview](https://docs.google.com/document/d/1Gvoy-4-_305eKA3BszSJPyQpVuOmsV4hLmKKKOxmEUk/edit?usp=sharing) doc.
- Added `README.md` and `CLAUDE.md`.
- Added `applications/` with one file per app: scoretility, routetility, contractility, membertility, tmtility, logmon, wordpress-docker.
- Added `framework/overview.md` covering the shared loutilities/Flask/DataTables/d3js stack.
- Added `infrastructure/hosting.md` covering server hosting and dev tooling.
- Added per-folder `README.md` index files for `applications/`, `framework/`, and `infrastructure/`.
- Set repo description on GitHub to "LLM wiki about FSRC Technology".
