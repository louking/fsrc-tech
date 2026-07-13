# CLAUDE.md

## What this repo is

This is **not a code project** — it's an LLM-oriented wiki documenting the FSRC Technology ecosystem: the technology that powers the Frederick Steeplechasers Running Club (FSRC), built and maintained by Lou King and licensed to the club for free use in perpetuity. Changes are markdown edits; the only "build" is publishing the wiki itself (see below).

The wiki content is built with [Zensical](https://zensical.org) (the Material for MkDocs team's successor static site generator — migrated 2026-07-11 from `mkdocs`/`mkdocs-material`, see `CHANGELOG.md`) and published to `fsrc-tech.steeplechasers.org`. Config still lives in `mkdocs.yml` (repo root) — Zensical reads that file natively, no Zensical-specific config format — which points `docs_dir` at `docs/`: **all wiki content lives under `docs/`, not at the repo root.** This is a hard requirement inherited from MkDocs, not a style choice: `docs_dir` can't be the same directory as (or any ancestor of) `mkdocs.yml` itself, which originally surfaced as `Config value 'docs_dir': The 'docs_dir' should not be the parent directory of the config file`. The repo was restructured into this layout for exactly that reason — don't move content back to the root. The site now deploys automatically: `.github/workflows/deploy.yml` builds with Zensical and `scp`s the output to the server on every push to `main`, using a dedicated deploy user and a Caddy volume mount set up for exactly this. The full setup (and gotchas already hit along the way) lives in `local-notes/mkdocs-operational-setup.md`, git-ignored and not part of the published wiki.

`mkdocs.yml` also sets `extra_css: [stylesheets/extra.css]` (`docs/stylesheets/extra.css`) for small styling tweaks the Material theme/markdown alone can't do — currently just widening and 2-line-clamping a table's last column (`race-services/reports/README.md`'s Notes column, which was wrapping long free text into tall rows) via `.md-typeset table:not([class]) td:last-child` rules. It's scoped by `:last-child` rather than a class since markdown tables have no per-column hooks, and applies site-wide — fine while this is still the only table in the wiki, but reconsider the selector if a future table's last column shouldn't get the same treatment. No `nav:` is configured, so navigation auto-generates from the file tree, using each page's front-matter `title` (if set) in preference to its first heading.

## Structure

- `docs/` — everything Zensical publishes. `docs/README.md` is the wiki's actual homepage (content, not a GitHub landing page — see below).
  - `docs/applications/` — one file per app (scoretility, routetility, contractility, membertility, tmtility, logmon, wordpress-docker). Each covers what the app does, its repo, docs, and public/admin site URLs.
  - `docs/framework/` — the shared python/javascript framework (loutilities, Flask/Jinja2, DataTables, SQLAlchemy, d3js) that most of the apps are built on top of.
  - `docs/infrastructure/` — hosting/server details (DigitalOcean, Rocky Linux, docker-compose) and dev tooling.
  - `docs/race-services/` — the technical/operational side of FSRC's race timing: chip timing, backup timing, finish-line video, used both for outside races FSRC contracts to time and the low-key/informal races it offers its own members. The counterpart to contractility's contract/business side (which only covers the contracted-race half). Unlike `applications/`/`framework/`/`infrastructure/`, this covers third-party hardware/software (Trident, RaceDay Scoring, Blue Iris) that FSRC didn't build but depends on and configures.
  - `docs/org-tech/` — third-party tech supporting general club operations rather than race day specifically: Google Workspace for Nonprofits (digital identity/access policy), MailChimp (member comms, paid race-director promotion), Facebook and the in-progress migration to FSRC Community. Same "didn't build it, but depends on and configures it" pattern as `race-services/`, seeded from Google Docs rather than a code repo — see "Deepening a race-services or org-tech page" below.
  - `docs/GLOSSARY.md` — FSRC/domain-specific jargon (Grand Prix, Equalizer, interest, RunSignup, etc.), kept sorted alphabetically. Add a term here when a page uses club- or domain-specific vocabulary an LLM wouldn't already know; don't duplicate generic software terms, which are linked inline instead.
- `README.md` (repo root) — a short pointer for GitHub visitors to `docs/README.md`. It is **not** the wiki homepage; don't add real content here.
- `local-notes/` — git-ignored (see `.gitignore`) scratch space for Lou's personal operational reference docs (e.g. a deploy-setup checklist) that don't belong in the published wiki content. Not indexed by any `README.md`.

Keep new content organized by topic in these folders rather than as one flat pile of notes — that's the whole point of this repo.

Each topic folder under `docs/` has its own `README.md` acting as an index (linking to the files inside it), separate from `docs/README.md` (the overall wiki homepage) and the repo-root `README.md` (the GitHub pointer). When adding a new file to a folder, also add a link to it in that folder's `README.md`.

## Changelog

`CHANGELOG.md` tracks notable updates to this wiki (newest entries first). Add an entry there whenever you add or meaningfully change content.

## Deepening an app's (or framework's, or infrastructure component's) page

Several of the repos (e.g. `members`, `loutilities`, `caddy-docker`) have their own `CLAUDE.md` with real architecture detail, gotchas, and patterns — check for one before writing from scratch. Pull the high-value parts (tech stack, architecture notes, known gotchas, key classes/modules) into the corresponding page here, but link to the repo's own `CLAUDE.md`/docs as the authoritative source rather than duplicating it in full — it'll drift out of sync otherwise.

All the source repos are cloned locally as siblings of this repo, under `C:\Users\lking\Documents\Lou's Software\projects\<repo-name>` — check there first with Read/Glob/Grep rather than reaching for `gh api` or `WebFetch` on GitHub. Some are nested one level deeper (e.g. `caddy-docker` is actually at `...\projects\caddy-docker\caddy-docker`, mirroring this repo's own `fsrc-tech\fsrc-tech` nesting) rather than directly in `projects\`, so `Glob` for the repo name first if a direct path 404s. Only fall back to GitHub if a repo isn't present locally, or the task specifically needs the canonical remote state (e.g. checking what's actually pushed/merged). When a repo has no `CLAUDE.md`, reading the actual local source to find key classes/modules and their docstrings works well too — that's how `docs/framework/loutilities.md` was built. This pattern isn't limited to `docs/applications/`/`docs/framework/` pages — `docs/infrastructure/caddy.md` was built the same way from the `caddy-docker` repo.

## Deepening a race-services or org-tech page

`docs/race-services/` and `docs/org-tech/` pages document third-party hardware/software/services (Trident, RaceDay Scoring, Blue Iris, Google Workspace, MailChimp, Facebook, etc.) that FSRC didn't build, so there's no repo `CLAUDE.md` or source code to pull from. Instead these are seeded from Lou's own operational Google Docs/Sheets/Slides — find and link the source doc the same way `docs/race-services/timing.md` links [Chip Timing Configuration and Operation](https://docs.google.com/document/d/1utAcupx2zqPyF6dONEBtDsQsB651qulztP_W2txLqsc/edit?usp=sharing) and the [Chip/Scoring RaceDay Scoring Checklist](https://docs.google.com/spreadsheets/d/1GcDuB508jKu_kdMXZ8VhzIaZk6LE2l4qIxwpstLUKIc/edit?usp=sharing), or `docs/org-tech/README.md` links the [Digital Access and Identity Policy](https://docs.google.com/document/d/1dxsT7itCPQ_iHNLq_rNajqF9Ry72GxwGo9QmAflob4U/preview) and the [member communications survey](https://docs.google.com/presentation/d/1d6UbXKCqowknwfKrcoBn2PuCpGTy01scSH2YXnBV2VY/preview) — condensing rather than duplicating verbatim in either case.

**Fetching a Google Doc/Sheet's actual content**: if a `mcp__claude_ai_Google_Drive__*` connector is available in the session (check the deferred-tools list), prefer it over `WebFetch` — `read_file_content`/`download_file_content` return the real body (Docs and Sheets both), and `get_file_metadata` gives the clean title for citing it, without any of the auth/chrome/link-stripping issues below. Only fall back to `WebFetch` when that connector isn't available: fetching a `docs.google.com/document/d/<id>/edit` URL directly only returns the editor chrome, not the body. Use the export endpoint instead — `.../export?format=txt` (follow the redirect it returns to a `googleusercontent.com` URL) gives the plain body text, but **silently strips all hyperlinks**, leaving only visible anchor text (e.g. a "References" list becomes bare titles with no way to tell they were ever links). If the doc has citation/reference links worth preserving, re-fetch with `format=html` instead and ask for the `href`/`q=` targets explicitly — that's how the real URLs for `docs/race-services/timing.md`'s References section were recovered after the txt export had silently dropped them.

**Checking a source doc's sharing visibility**: before deciding how much to condense vs. link out, check `get_file_permissions` on the doc — don't assume it's access-restricted just because it isn't public knowledge. `docs/race-services/timing.md`'s RaceDay Scoring checklist section had drifted into a near-verbatim, field-by-field transcription of the [Chip/Scoring RaceDay Scoring Checklist](https://docs.google.com/spreadsheets/d/1GcDuB508jKu_kdMXZ8VhzIaZk6LE2l4qIxwpstLUKIc/edit?usp=sharing) sheet; the assumption going in was that the sheet's visibility was limited, but `get_file_permissions` showed it's actually shared "anyone with the link, reader" — confirming it was safe to condense the wiki section down to a pointer at the sheet plus a handful of stable config values, rather than keep it fully duplicated.

**Verifying deep-links (`#anchor`) into external docs**: `WebFetch`'s summarized answer for "what's the id of section X" can be an inferred/guessed slug (e.g. "following standard heading-to-id convention") rather than one actually read off the page — don't trust it at face value for anchors. Download the real HTML (`curl`/Bash into the scratchpad dir) and grep for `id="..."` / `<section id=...>` directly. This also matters because external doc anchors go stale silently — no 404, the link just lands at the top of the page — when the target repo restructures its docs (e.g. `tm-csv-connector` dropped its "Scanner Connected"/"Scanner Not Connected" subsections, breaking `docs/race-services/timing.md`'s link to `#tmtility-operation-scanner-connected`). When a linked repo's docs are known to have changed, re-verify anchors rather than assuming they still resolve.

## Terminology

Prefer FSRC-facing product names over the underlying framework/platform name: "FSRC Community" or "FSRC Community forum", not "Discourse"; "steeplechasers.org", not "WordPress". The framework name is fine (and often necessary) when the content is actually about the technical integration — API details, config keys, architecture notes, gotchas — just not as the everyday way of referring to the thing. See `docs/GLOSSARY.md` for the FSRC Community entry.

## Voice: don't attribute to "Lou" inside technical content

Architecture notes and gotcha bullets (in `docs/applications/`, `docs/framework/`, `docs/infrastructure/`, `docs/race-services/`, `docs/org-tech/`) are written in passive/technical voice describing the system's behavior — no "Lou added X" / "Lou fixed Y". Reserve naming Lou by name for framing-level content only: the `README.md`/`CLAUDE.md` statements of ecosystem ownership, and `CHANGELOG.md` entries about his personal tooling/workflow choices (e.g. "Cursor is now Lou's primary dev editor").

## License

Written content here is under `LICENSE` (CC BY-SA 4.0) — a documentation license, not a software one. This is separate from and does not apply to the application source code in the individual app repos, which have their own (Apache-2.0) licensing.

## Git commit messages

In this environment, `git commit -m "$(cat <<'EOF' ... EOF)"` (heredoc-in-command-substitution) has hung indefinitely rather than completing. Write the message to a temp file and use `git commit -F <file>` instead — reliable, and sidesteps quoting/heredoc issues entirely.

## Pushing changes to `.github/workflows/`

GitHub rejects a push that creates or updates any file under `.github/workflows/` (e.g.
`deploy.yml`) if it's authenticated with a Personal Access Token that lacks the `workflow`
scope — a restriction independent of normal repo write access:

```
! [remote rejected] main -> main (refusing to allow a Personal Access Token to create or
update workflow `.github/workflows/deploy.yml` without `workflow` scope)
```

Fix depends on the credential helper in use: `gh auth refresh -h github.com -s workflow` if
`gh` is the credential helper; otherwise edit the token at `github.com/settings/tokens` (classic)
or `github.com/settings/tokens?type=beta` (fine-grained, needs "Workflows: Read and write") to
add the scope, then retry the push — no other change needed, the token value itself doesn't
change.

## Slow git commands after a local `.venv` exists

Once a `.venv/` (e.g. for Zensical, see below) exists in this repo, plain `git status`/`git mv` calls have taken several minutes instead of being near-instant — observed repeatedly in this environment, cause unconfirmed (not OneDrive — this path isn't under OneDrive, and `Documents` isn't redirected there — so likely antivirus/Defender or some other real-time scanner reacting to the venv's thousands of small files, even though `.venv/` is gitignored). Don't assume a hung git command is broken or retry it; run it with `run_in_background` and wait rather than killing/repeating it.

## Markdown checklists

Use unordered list markers for task-list checkboxes — `- [ ]` / `* [ ]`, not ordered markers (`1. [ ]`). This is the portable GFM/CommonMark form and is confirmed to render correctly in Obsidian; rely on list order rather than manually-numbered titles if sequence matters.

Cursor's own Markdown preview, however, was seen not rendering even correctly-formed `- [ ]` checkboxes (`local-notes/mkdocs-operational-setup.md`) while Obsidian rendered the same file fine — so a checklist not checking in Cursor isn't necessarily a syntax problem in the file; it may be a Cursor preview/extension limitation. Verify in a second renderer (Obsidian, GitHub) before assuming the markdown itself is wrong.

## Source material

The actual application code lives in separate repos under https://github.com/louking/ (rrwebapp, runningroutes, contracts, members, tm-csv-connector, logmon, loutilities, wordpress-docker, caddy-docker). This repo only holds descriptive/reference content, not the app source.

The initial content was seeded from Lou's own overview doc, [FSRC Technology Software Overview](https://docs.google.com/document/d/1Gvoy-4-_305eKA3BszSJPyQpVuOmsV4hLmKKKOxmEUk/edit?usp=sharing), which he uses as the starting point when explaining the FSRC tech stack to someone new.

`docs/race-services/` and `docs/org-tech/` are exceptions to the "code repos + one overview doc" pattern above: they document third-party tech FSRC configures but didn't build (race-day tech, and general club-operations tech respectively), seeded from separate operational Google Docs/Sheets/Slides Lou maintains (not code repos) — see the "Deepening a race-services or org-tech page" section above. This realizes the broader expansion agreed 2026-07-07 — covering other third-party tech FSRC depends on beyond Lou's own repos — which `docs/org-tech/` (added 2026-07-13) was the first concrete instance of; further third-party dependencies (e.g. more Google Workspace detail) should extend `docs/org-tech/` rather than reopen the folder-structure question.
