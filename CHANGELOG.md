# Changelog

A running log of notable updates to this wiki. Newest entries at the top.

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
