# Changelog

A running log of notable updates to this wiki. Newest entries at the top.

## 2026-07-07

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
