# membertility

Club management app with several modules.

- Admin site: https://members.loutilities.com/admin
- Repo: https://github.com/louking/members
- Docs: https://docs.members.loutilities.com/en/latest/
- Repo's own `CLAUDE.md` (authoritative, deeper detail than this page): https://github.com/louking/members/blob/master/CLAUDE.md

## Modules

- **organization** — maintains the club leadership organization, committees, etc.
- **tasks** — for people in certain positions, manages tasks they need to do, mostly for onboarding (e.g., read document x, fill out this form, take this online course); manages racing team contracts; tracks coach training
- **meetings** — manages club meetings (board meetings, etc.), maintaining RSVPs, status reports, agenda, minutes, action items, motions, and votes (e-voting)
- **membership** — caches club membership information as collected from the registration service (RunSignup); provides public views into membership status and statistics; uses d3js to draw membership statistics
- **racing team** — provides user forms to (1) collect applications, and (2) collect information about races run and volunteerism; provides a database for admins to track racing team members over time
- **awards** — tracks race awards and associated competitions (RunSignup-based)
- **community** — manages the FSRC Community forum (built on Discourse): group sync (from RunSignup races/clubs or internal position tags), event topic creation, a tag-filterable calendar (`.ics`) feed, forum taxonomy export, and a pending-review notifier (Discourse's own category-group-moderation feature never emails/PMs moderators, only shows an on-site badge)
- **super admin** — system-level configuration: SSO/auth, email, file storage, user roles, interest attributes

## Tech stack

- Backend: Python 3.12, Flask 3.0, SQLAlchemy 2.0, Flask-Security-Too, Flask-Migrate (Alembic)
- Database: MySQL 8.0 (via PyMySQL driver — see gotcha below)
- Frontend: Jinja2 templates, Webassets (CSS/JS bundling), Bootstrap
- Infrastructure: Docker + Docker Compose, Nginx reverse proxy, Gunicorn
- External integrations: Google Workspace API, MailChimp, RunSignup API, Discourse API, Mailgun (msmtp)
- Shared libraries: [loutilities](../framework/overview.md) (custom framework), `runtilities`

## Architecture notes

- **Multi-tenancy**: URL structure is `/app/<interest>/admin/...`. The `interest` (e.g., which club/series) is stashed in Flask's `g` via a URL preprocessor; views use a `localinterest()` helper to read it.
- **Blueprints**: admin-facing views (`views/admin/`) are separate from member-facing views (`views/frontend/`).
- **Config**: INI-format files under `config/`, mounted read-only in Docker; secrets come from Docker secrets (`/run/secrets/`). Values are run through `eval()` by `loutilities.configparser`, so booleans/ints/dicts/lists arrive as native Python types, not strings.
- **CLI**: four Click command groups run as `flask <group> <command>` — `members`, `membership` (e.g., MailChimp sync), `task`, `community` (FSRC Community group sync, event import, taxonomy export).
- **Calendar feed**: a single endpoint, `/<interest>/calendars/events.ics`, served on-demand by the Flask app itself (not static files, not cron). `tags=<tag>[,<tag>...]` filters to events carrying any of the listed tags (union; omitted = all events); `year=`/`from=`/`to=` bound the date window (`from=today` is a supported literal; omitted `to` means open-ended/all future). Compiled ICS bytes are cached per `(interest, tags, from, to)` (15 min TTL). The older per-series `<series>.ics` URLs, and the disk-writing `filter-calendar` CLI command and `CALENDAR_TAG_GROUPS_<INTEREST>` config key that backed them, were removed outright rather than redirected/deprecated — nothing else in the repo (cron, Nginx) referenced them.
- **Pending-review notifier**: `flask community notify-pending-reviews` polls the Discourse Admin API's review queue per category and PMs each category's moderator group(s) once an item has been pending past a threshold, escalating with repeat PMs if still unresolved after a longer window. Not event-specific — it surfaces any pending reviewable in the category (flagged posts, new-user posts, event topics, etc.), since Discourse's review queue has no way to filter by "is this an event." Moderator group(s) are read live from Discourse per category — `category.topic_posting_review_group_ids` / `reply_posting_review_group_ids` on `GET /c/{id}/show.json` (group ids, not names; resolved via `GET /groups.json`) — not the `reviewable_by_group_name`-style field the "category-group-moderation" feature name suggests, which doesn't exist on this Discourse version. Notified-state lives in a DB table (not a file), scoped per interest; intended to run every 15 minutes via cron, and deliberately silent on routine runs (nothing pending) so the frequent cadence doesn't spam cron mail. The PM's sending account is configurable per interest (falls back to the shared admin account used for group syncs/invites), same override pattern as the event-import and calendar-feed accounts.

## Gotchas worth knowing

- **JS assets are shadowed by a Docker volume mount** to an external `js-common` directory (shared across apps). Editing `app/src/members/static/js/` in the repo directly has no effect on the running container.
- **MySQL 8 + Alpine containers**: `mysqlclient` triggers a TLS/SSL certificate verification failure (Alpine's MariaDB Connector/C defaults to SSL). Fix is to switch to the pure-Python **PyMySQL** driver (`mysql+pymysql://` URI scheme) instead of patching SSL settings.
- **`AwardsAwardee.prev_awardee` self-referential relationship is inferred backwards**: the relationship is missing `remote_side=[id]`, which SQLAlchemy needs to disambiguate a self-referential FK's direction. Left as-is deliberately ([members#710](https://github.com/louking/members/issues/710)) rather than fixed in place — applying the "correct" relationship would silently null out the pickup-warning UI for every existing historical award pair without a data migration to repoint them first.
- **Community module's Discourse rate limiting is cross-process**: the `_RateLimiter`'s call history is persisted to a shared Docker volume (`community-locks`, mounted at `/locks` in both the `app` and `crond` containers) rather than in-process or `/tmp`, since throttling has to hold across separate containers/filesystems making API calls independently. A fresh volume can itself be created root-owned, silently blocking the limiter's writes the first time — worth checking if throttling appears not to be taking effect after a volume reset.
- **Discourse API quirks**: booleans must be passed as lowercase strings (`'true'`/`'false'`, not Python `True`), several endpoints return unexpectedly shaped data (e.g., `site_settings.json` returns a list; topic `tags` may be strings or objects), and category group permissions aren't exposed by any Discourse REST endpoint — they're fetched via a Data Explorer SQL query instead. `GET /review.json`'s pagination shape is undocumented and unsafe to page speculatively: an incrementing `page` param can keep returning the same non-empty page indefinitely instead of eventually coming back empty, burning through the API rate limit — the pending-review notifier uses a single bounded request instead of looping. A reviewable's own record has no reader-meaningful identity (its `id` is a bare number, `created_at` is UTC) — the notifier instead pulls the post/topic title and submitter username out of the response's sibling data and converts the timestamp to the container's local timezone (from its `TZ` env var) for the PM.
- **`pytest` test suite** covers `helpers.py` (RunSignUp client factories, position/date utility functions) against SQLite, run via `bareapp`/`bare_dbapp` fixtures rather than the full `create_app()` — `create_app()` unconditionally queries a table that doesn't exist yet at that point in test setup, the same ordering issue documented for `contracts`. Four `awards_*` tables use a MySQL-only column default and are excluded from the test database's `create_all()`. `loutilities` also has its own separate tests (`python -m pytest tests/`).
- **`setuptools>=83` dropped `pkg_resources` entirely**, breaking any dependency that still does an unconditional `from pkg_resources import ...`. `FormEncode==2.0.1` (pulled in by `loutilities.tables`) is affected; `FormEncode>=2.1.1` uses `importlib.metadata` instead and doesn't need `setuptools` pinned back down.
- **A local `.venv` at the repo root is treated as the source of truth for `requirements.txt`** when the Docker build's `pip install` hits a resolver conflict — reconcile against `pip list --format=freeze` from that venv rather than guessing a new pin.

For full detail on any of the above (directory layout, all CLI commands, deployment via Fabric, Discourse client patterns), see the repo's own `CLAUDE.md` linked above — treat it as the authoritative, actively-maintained source rather than duplicating it here.
