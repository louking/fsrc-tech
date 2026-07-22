---
name: wiki-lint
description: Health-check pass over the fsrc-tech wiki — broken internal links, orphaned pages, unused glossary terms, and (with `deep`) staleness against source repos/docs. Use when the user asks to lint, audit, or check the health of this wiki, or references the "Lint" operation from the wiki's guiding gist.
user-invocable: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash(find *)
  - WebFetch
  - mcp__claude_ai_Google_Drive__get_file_metadata
---

# /wiki-lint — fsrc-tech wiki health check

This is the **Lint** operation from this wiki's guiding pattern (see
`CLAUDE.md`'s "Guiding pattern for wiki maintenance" section and the
[Karpathy LLM-wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
it references): Ingest and Query already have workflows in `CLAUDE.md`; this
skill is the periodic pass that catches drift between them.

**Report findings, don't silently fix them.** Present a numbered list of
issues found (file, problem, suggested fix) and ask which to act on, unless
the user has already said "and fix what you find" up front. This is a read
audit by default.

Arguments passed: `$ARGUMENTS` — empty for a quick (mechanical) pass, `deep`
to add the slower checks described below.

---

## Quick pass (always run)

1. **Broken internal links.** Grep every `docs/**/*.md` for markdown links
   `[...](...)` whose target is a relative path (not `http(s)://`, not a bare
   `#anchor`). Resolve each relative to the linking file's own directory and
   confirm the target file exists. Flag any that don't. Watch for the two
   real gotchas already on file in this repo: relative paths that assume
   `docs_dir` (they're relative to the *linking file*, not the repo root —
   e.g. `../GLOSSARY.md` from a topic folder, not `docs/GLOSSARY.md`), and
   `#anchor` fragments into *other* local pages (confirm the heading still
   exists, headings can get renamed).

2. **Orphaned pages.** For each topic folder (`applications/`, `framework/`,
   `infrastructure/`, `org-tech/`, `race-services/`, `race-services/reports/`),
   list its `.md` files (Glob) and confirm each one (other than the folder's
   own `README.md`) is linked from that folder's `README.md`. Then confirm
   each folder's `README.md` is reachable from `docs/README.md` (directly or
   transitively). Flag anything unreachable — per `CLAUDE.md`'s Structure
   section, every new file is supposed to get a link in its folder's index.

3. **Unused glossary entries.** For each term heading in `docs/GLOSSARY.md`,
   Grep the rest of `docs/` for a reference to it (a link to
   `GLOSSARY.md#<anchor>`, or a highlighted/linked mention of the term).
   Flag terms defined but never linked from anywhere — either a page should
   link it, or the entry may no longer be needed.

4. **CHANGELOG hygiene.** Spot-check: pick a few `docs/**/*.md` files with
   the most recent git modification times (`git log -1 --format=%cd -- <file>`)
   and confirm each has a matching `CHANGELOG.md` entry at or after that
   date. Flag content changes that never got logged.

## Deep pass (`deep` argument only — slower, needs judgment)

5. **Source-repo drift.** For each `docs/applications/`, `docs/framework/`,
   and `docs/infrastructure/` page that names a source repo, check whether
   that repo (cloned locally under `Lou's Software\projects\<repo-name>`,
   per `CLAUDE.md`'s "Deepening..." section) has `CLAUDE.md` content not yet
   reflected here — i.e. re-run the existing Ingest check rather than a new
   mechanism.

6. **External anchor rot.** Re-verify any `#anchor` link into another repo's
   docs or a Google Doc (per `CLAUDE.md`'s "Verifying deep-links" gotcha —
   these go stale silently, no 404, the link just lands at the top of the
   page). Fetch the real target HTML and grep for the `id=`/`<section id=`
   the link expects.

7. **Google Doc/Sheet staleness.** For each `docs/race-services/` or
   `docs/org-tech/` page sourced from a Google Doc/Sheet, call
   `get_file_metadata` on the source doc (if the Drive connector is
   available this session) and compare `modifiedTime` against the date the
   wiki page was last touched (`CHANGELOG.md` or git log). Flag docs edited
   since the wiki last pulled from them.

8. **Contradictions.** While reading pages for the checks above, note any
   place two pages describe the same fact (a URL, a config value, an
   architecture claim) differently. This one is inherently manual — flag
   what you notice, don't try to exhaustively diff every page pair.

---

## Output

End with a short numbered findings list (or "no issues found" per category).
For each finding: file, what's wrong, one-line suggested fix. Don't create a
report file — this is a conversational audit, not a wiki page. If the user
asks to act on a finding, apply it as a normal edit (and follow `CLAUDE.md`'s
`CHANGELOG.md` convention if the fix is a real content change).
