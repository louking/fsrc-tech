# wordpress-docker

Provides a docker container for WordPress, used for steeplechasers.org.

- Repo: https://github.com/louking/wordpress-docker
- No repo-level `CLAUDE.md` — this page is built directly from `docker-compose.yml`/`fabfile.py`.

Unlike the other xtility apps, this isn't a custom Flask app — it's a `docker-compose` stack wrapping the stock WordPress image, used to run [steeplechasers.org](../GLOSSARY.md) itself (the club's public website, as distinct from the FSRC Community forum).

## Stack

- **`wordpress`** — custom image (`louking/${APP_NAME}-wordpress`) built from the official WordPress image (`WORDPRESS_VER`/`PHP_VER` build args); serves the site on `${APP_PORT}`.
- **`db`** — MySQL (`MYSQL_VER`), using `--default-authentication-plugin=mysql_native_password` for compatibility with the WordPress client libraries.
- **`crond`** — a separate Alpine-based container running scheduled DB backups (`docker-library` cron pattern), with `PROD`/`SANDBOX`/`DEV` environment flags controlling behavior per deployment target.
- Volumes: `wordpress` (site files), `db` (MySQL data), plus a read-only bind mount for `leaderboards` content and shared host log volumes.
- Secrets: `root-password` and `appdb-password`, mounted from `config/db/*.txt` as Docker secrets (not environment variables).

## Deployment

Deployed via Fabric (`fabfile.py`), same pattern as the other xtility apps: `fab -H <target-host> deploy [prod|sandbox]`, optionally with `--branchname=<branch>`. The task pulls `docker-compose.yml` fresh from GitHub onto the target host, then runs `docker compose pull && docker compose up -d` — the deployed compose file always comes from the repo, not a locally-edited copy.

## Gotchas worth knowing

- **MySQL 8.4 auth plugin**: the compose file explicitly pins `--default-authentication-plugin=mysql_native_password`, since MySQL 8.4 changed the default auth plugin in a way that breaks older WordPress DB clients — don't remove that flag when bumping `MYSQL_VER`.
- **Backups**: `crond` mounts `${BACKUP_FOLDER_HOST}/${APP_DATABASE}:/backup` — check that host path exists on a new server before first deploy.
