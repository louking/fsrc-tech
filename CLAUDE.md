# CLAUDE.md

## What this repo is

This is **not a code project** — it's an LLM-oriented wiki documenting the FSRC Technology ecosystem: the software Lou King has built and maintains for the Frederick Steeplechasers Running Club (FSRC). There is no build, test, or lint step here; changes are just markdown edits.

## Structure

- `applications/` — one file per app (scoretility, routetility, contractility, membertility, tmtility, logmon, wordpress-docker). Each covers what the app does, its repo, docs, and public/admin site URLs.
- `framework/` — the shared python/javascript framework (loutilities, Flask/Jinja2, DataTables, SQLAlchemy, d3js) that most of the apps are built on top of.
- `infrastructure/` — hosting/server details (DigitalOcean, Rocky Linux, docker-compose) and dev tooling.
- `GLOSSARY.md` — FSRC/domain-specific jargon (Grand Prix, Equalizer, interest, RunSignup, etc.), kept sorted alphabetically. Add a term here when a page uses club- or domain-specific vocabulary an LLM wouldn't already know; don't duplicate generic software terms, which are linked inline instead.

Keep new content organized by topic in these folders rather than as one flat pile of notes — that's the whole point of this repo.

Each topic folder has its own `README.md` acting as an index (linking to the files inside it), separate from the root `README.md`. When adding a new file to a folder, also add a link to it in that folder's `README.md`.

## Changelog

`CHANGELOG.md` tracks notable updates to this wiki (newest entries first). Add an entry there whenever you add or meaningfully change content.

## Deepening an app's (or framework's) page

Several of the repos (e.g. `members`, `loutilities`) have their own `CLAUDE.md` with real architecture detail, gotchas, and patterns — check for one before writing from scratch. Pull the high-value parts (tech stack, architecture notes, known gotchas, key classes/modules) into the corresponding page here, but link to the repo's own `CLAUDE.md`/docs as the authoritative source rather than duplicating it in full — it'll drift out of sync otherwise. When a repo has no `CLAUDE.md`, reading the actual source (e.g. via `gh api repos/<owner>/<repo>/contents/<path>` or `WebFetch` on the raw GitHub URL) to find key classes/modules and their docstrings works well too — that's how `framework/loutilities.md` was built.

## Terminology

Prefer FSRC-facing product names over the underlying framework/platform name: "FSRC Community" or "FSRC Community forum", not "Discourse"; "steeplechasers.org", not "WordPress". The framework name is fine (and often necessary) when the content is actually about the technical integration — API details, config keys, architecture notes, gotchas — just not as the everyday way of referring to the thing. See `GLOSSARY.md` for the FSRC Community entry.

## License

Written content here is under `LICENSE` (CC BY-SA 4.0) — a documentation license, not a software one. This is separate from and does not apply to the application source code in the individual app repos, which have their own (Apache-2.0) licensing.

## Git commit messages

In this environment, `git commit -m "$(cat <<'EOF' ... EOF)"` (heredoc-in-command-substitution) has hung indefinitely rather than completing. Write the message to a temp file and use `git commit -F <file>` instead — reliable, and sidesteps quoting/heredoc issues entirely.

## Source material

The actual application code lives in separate repos under https://github.com/louking/ (rrwebapp, runningroutes, contracts, members, tm-csv-connector, logmon, loutilities, wordpress-docker). This repo only holds descriptive/reference content, not the app source.

The initial content was seeded from Lou's own overview doc, [FSRC Technology Software Overview](https://docs.google.com/document/d/1Gvoy-4-_305eKA3BszSJPyQpVuOmsV4hLmKKKOxmEUk/edit?usp=sharing), which he uses as the starting point when explaining the FSRC tech stack to someone new.
