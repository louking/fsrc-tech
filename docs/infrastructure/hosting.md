# Hosting & infrastructure

All apps and the website run as [docker-compose](https://docs.docker.com/compose/) apps, to facilitate development and migration to a new server if necessary.

- Current server: [DigitalOcean](https://www.digitalocean.com/), running [Rocky Linux](https://rockylinux.org/) 9
- Previous server: CentOS 7 (reached end of life, which precipitated the migration to Docker)
- Last server upgrade completed: Feb 20, 2025
- [Caddy](caddy.md) runs as its own container and acts as the edge reverse proxy in front of all the app containers, handling TLS/domain routing for the whole server

## Development

- Development is done using [Cursor](https://cursor.com/blog/cursor-3), which is based on [Visual Studio Code](https://code.visualstudio.com/)
- All software is maintained on [GitHub](https://github.com/)
- [Caddy](caddy.md) is also used locally, proxying `*.localhost` domains to each app's port, so local dev URLs mirror the production domain structure
- Lou's own development process notes (server/account setup, database migration handling, Python package release procedure, Docker, HTTPS, virtual host setup) are maintained separately as a readthedocs doc set at [process.loutilities.com](https://process.loutilities.com/) — it also covers development-environment setup and `loutilities`-specific coding patterns (see [loutilities.md](../framework/loutilities.md)), not just hosting. Caution: some of it predates the Docker/Rocky Linux migration above and may be out of date
