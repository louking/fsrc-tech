# Changelog

A running log of notable updates to this wiki. Newest entries at the top.

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
