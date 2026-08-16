# wordpress-docker

Provides a docker container for WordPress, used for steeplechasers.org.

- Repo: https://github.com/louking/wordpress-docker
- Repo's own `CLAUDE.md` (authoritative, deeper detail than this page): https://github.com/louking/wordpress-docker/blob/main/CLAUDE.md

Unlike the other xtility apps, this isn't a custom Flask app — it's a `docker-compose` stack wrapping the stock WordPress image, used to run [steeplechasers.org](../GLOSSARY.md) itself (the club's public website, as distinct from the FSRC Community forum).

## Stack

- **`wordpress`** — custom image (`louking/${APP_NAME}-wordpress`) built from the official WordPress image (`WORDPRESS_VER`/`PHP_VER` build args), adding vim/bash and PHP config overrides (`/usr/local/etc/php/conf.d/`); serves the site on `${APP_PORT}`. A read-only bind mount brings in externally-updated `leaderboards/` content.
- **`db`** — MySQL (`MYSQL_VER`), using `--default-authentication-plugin=mysql_native_password` for compatibility with the WordPress client libraries; binlogs capped at 3 days to prevent unbounded growth.
- **`crond`** — a separate Alpine-based container running scheduled DB backups (`mariadb-dump`, gzipped) as root, with `PROD`/`SANDBOX`/`DEV` environment flags controlling behavior per deployment target.
- Volumes: `wordpress` (site files), `db` (MySQL data), plus the `leaderboards` bind mount and shared host log volumes.
- Secrets: `root-password` and `appdb-password`, mounted from `config/db/*.txt` as Docker secrets (not environment variables).

## Deployment

Deployed via Fabric (`fabfile.py`), same pattern as the other xtility apps: `fab -H <target-host> deploy [prod|sandbox]`, optionally with `--branchname=<branch>`. The task pulls `docker-compose.yml` fresh from GitHub onto the target host, then runs `docker compose pull && docker compose up -d` — the deployed compose file always comes from the repo, not a locally-edited copy, so an image change (Dockerfile, ini file) needs its own image build/push under the version tags the target branch's `.env` references. Fabric's own `APP_NAME` (`'steeplechasers.org'`, the remote deploy directory name) is unrelated to `.env`'s `APP_NAME=wordpress-docker` (which feeds Docker image tags) — don't assume the two identifiers should match.

## Gotchas worth knowing

- **MySQL 8.4 auth plugin**: the compose file explicitly pins `--default-authentication-plugin=mysql_native_password`, since MySQL 8.4 changed the default auth plugin in a way that breaks older WordPress DB clients — don't remove that flag when bumping `MYSQL_VER`.
- **Backups**: `crond` mounts `${BACKUP_FOLDER_HOST}/${APP_DATABASE}:/backup` — check that host path exists on a new server before first deploy.
- **Silent PHP-ini glob miss**: the Dockerfile's `COPY` for PHP overrides matches `*php.ini`, not `*.ini` — it only picks up filenames literally *ending* in `php.ini` (e.g. `zz-all-in-one-wp-migration-php.ini`). A new override like `zz-upload-max-filesize.ini` is silently excluded from the image with no build error; the setting just never takes effect unless the file is renamed to end in `php.ini` or the glob is widened.
- **Environment-gated cron jobs, not environment-specific images**: the same `crond` image runs in every environment — which jobs actually fire is controlled by `PROD`/`SANDBOX`/`DEV` env vars, checked inline per crontab line (`test "$PROD" && ...`). Follow that pattern when adding a job rather than branching at the image/compose level.
