# lmtility (logmon)

Monitors application logs and server utilization.

- Repo: https://github.com/louking/logmon
- Docs: https://github.com/louking/logmon/blob/main/README.md
- Repo's own `CLAUDE.md` (authoritative, deeper detail than this page): https://github.com/louking/logmon/blob/main/CLAUDE.md
- Monitors application logs for exceptions; sends email when an exception occurs, but throttles sending too much email
- Monitors access logs for "bad actor" web accesses
- Monitors disk usage for volumes and within docker volumes, and memory/swap utilization
- Shows CPU utilization via the DigitalOcean API
- Displays SNS/SES webhook notifications alongside log events

## Tech stack

- Backend: Flask, SQLAlchemy, Flask-Migrate (Alembic)
- Database: MySQL (two binds — see below)
- Cache/live-tail buffer: Redis
- Shared libraries: [loutilities](../framework/loutilities.md) (`db`, shared user/role models)

## Architecture notes

- **Services**: `app` (web UI, SNS webhook, JSON API — scaled freely), `follower` (single-instance daemon tailing logs and collecting disk/memory stats — must stay single-instance or tailing duplicates), `redis` (shared live-tail buffer between `app` and `follower`), `db` (MySQL), `crond` (backups, log pruning), `web` (nginx + phpMyAdmin). `app` and `follower` share the same Docker image but run different commands.
- **Two entry points**: `app_server.py` (WSGI/gunicorn) and `app.py` (Flask CLI only, incl. Flask-Migrate); both call `logmon.create_app()`.
- **Background threads** (follower container only, started by `flask run-follower`): `FollowerManager` (rescans monitored apps every 30s, spawns a `FileFollower` thread per log file — assembles multi-line tracebacks, parses access log lines), `DiskMonitor` (runs `df -P`/`docker system df` on an interval, alerts past a threshold), `MemMonitor` (reads `/proc/meminfo` on an interval, alerts on swap usage — note this reflects host kernel memory, not the container's cgroup limit).
- **Two databases**: default bind (`LogEvent`, `AccessEvent`, `AlertSuppression`, `SnsNotification`, `DiskSnapshot`, `MemSnapshot`) and a `users` bind (`User`, `Role`, shared with other apps and managed externally); Flask-Migrate only runs against the default DB.
- **Alert suppression**: log, disk, and swap alerts all share one `AlertSuppression` table, keyed by `app_name`/`exception_type`, so the same suppress-and-reset logic covers all three alert types.
- **Access analysis**: bad-actor detection queries `AccessEvent` directly (no separate aggregation job); CIDR-to-country data is downloaded from ipdeny.com on startup, and private/RFC-1918 ranges are excluded automatically.
- **Adding a monitored app**: requires updating both `docker-compose.override.yml` (bind mount) and `config/logapps.yml` (log directory entry) — picked up within 30 seconds, no restart needed.

## Gotchas worth knowing

- Several config files are gitignored and must be copied from `.example` versions on a fresh checkout: `docker-compose.override.yml`, `config/logapps.yml`, `config/*.txt` (Docker secrets for Flask secret key, DB passwords, SMTP password, SNS webhook key).

For full detail (all CLI commands, service table, configuration loading), see the repo's own `CLAUDE.md` linked above.
