# This wiki

How this wiki itself is organized, built, and published. Source: [louking/fsrc-tech](https://github.com/louking/fsrc-tech). The repo's own [`CLAUDE.md`](https://github.com/louking/fsrc-tech/blob/main/CLAUDE.md) is the authoritative source for maintenance conventions (voice, cross-referencing, ingest/lint workflow); this page summarizes the structure and build.

## Structure

- All published content lives under `docs/`; anything at the repo root (`README.md`, `CLAUDE.md`, `CHANGELOG.md`, `LICENSE`) is repo-only and never reaches the live site
- `docs/README.md` is the wiki homepage. The repo-root `README.md` is only a pointer for GitHub visitors
- Content is organized by topic folder — `applications/`, `framework/`, `infrastructure/`, `race-services/`, `org-tech/` — plus `GLOSSARY.md` at the top level
- Each topic folder has its own `README.md` acting as that folder's index; a new page gets a link there
- `docs/stylesheets/extra.css` holds small styling tweaks the Material theme can't do from markdown alone (currently a site-wide rule widening and 2-line-clamping a table's last column)
- `CHANGELOG.md` (repo root) records notable content changes, newest first
- `.claude/skills/` holds Claude Code skills for maintaining the wiki (e.g. `wiki-lint`); not published

## Build

- Built with [Zensical](https://zensical.org), the Material for MkDocs team's successor static site generator. The only dependency is `zensical`, pinned to an exact version in `requirements.txt` so CI and local previews build identically
- Configuration is `mkdocs.yml` at the repo root, which Zensical reads natively: site name/URL, the `material` theme, `extra_css`, `docs_dir: docs`, and `site_dir: site`
- No `nav:` is configured — navigation is generated from the file tree, titling each page by its front-matter `title` if set, otherwise its first heading
- Content must stay under `docs/` rather than the repo root: `docs_dir` can't be the directory containing `mkdocs.yml` (or any ancestor of it), so a root-level layout fails config validation

### Local preview

From the repo root, in a virtual environment:

```bash
pip install -r requirements.txt
zensical serve
```

`zensical build` writes the static site to `site/` (git-ignored), same as CI.

## Publishing

- Every push to `main` triggers `.github/workflows/deploy.yml`, which installs `requirements.txt`, runs `zensical build`, and copies `site/` to the server over `scp` as a dedicated deploy user
- On the server, [Caddy](caddy.md) serves the copied files as static content at [fsrc-tech.steeplechasers.org](https://fsrc-tech.steeplechasers.org) via a volume mount
- There is no manual publish step and no preview/staging site — merging to `main` is publishing

## Gotchas

- Pushing a change to anything under `.github/workflows/` requires a GitHub token with the `workflow` scope, independent of normal repo write access; without it the push is rejected outright
- The repo is public even though only `docs/` is published, so repo-root files (`CHANGELOG.md`, `CLAUDE.md`) should be written with the same care as wiki pages
- The repo is Windows-native, unlike most source repos, which have moved to WSL2 (see [hosting](hosting.md#development)). With a local `.venv/` present, `git` commands in the checkout have been seen to run for minutes rather than seconds
