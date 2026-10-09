# tmtility

Manages results received from a Time Machine manual race timer for input to scoring software like [RaceDay Scoring](https://info.runsignup.com/products/raceday/raceday-scoring/).

- Repo: https://github.com/louking/tm-csv-connector
- Docs: https://tm-csv-connector.readthedocs.io/en/latest/
- Repo's own `CLAUDE.md` (authoritative, deeper detail than this page): https://github.com/louking/tm-csv-connector/blob/master/CLAUDE.md
- Runs on a laptop, so there's no publicly accessible site
- Receives results from the [Time Machine](https://timemachine.org/) over Bluetooth, bib numbers scanned from participant bibs, and optionally chip reads from a Trident RFID reader
- Creates a CSV file which can be read by race scoring software (designed for and tested with RaceDay Scoring)
- Also runs in "simulator mode" so a prospective scoring operator can practice
- For chip-timing (Trident) theory, configuration, and race-day operation this app fits into, see [race-services/timing.md](../race-services/timing.md)

## Tech stack

- Backend: Flask, SQLAlchemy, Flask-Migrate (Alembic)
- Frontend: Jinja2, flask-assets bundles, WebSocket-driven live results view (`results.js`)
- Packaging: PyInstaller-built Windows client executables, distributed as zips and installed via NSSM as Windows services
- Shared libraries: [loutilities](../framework/loutilities.md) (`DbCrudApi` for table views)

## Architecture notes

- **Two operating modes**, toggled by `SIMULATION_MODE`: normal mode auto-logs in a default user, shows public views only, and talks to real hardware; simulation mode exposes full multi-user login and `/admin/*` routes, replaying recorded events for operator training.
- **Three external client processes** run as Windows services (not in the Docker container, since they need direct access to USB/Bluetooth hardware): `tm-reader-client` (Time Machine Bluetooth), `barcode-scanner-client` (bib scanner), `trident-reader-client` (Trident RFID telnet connection). Each connects to the Flask backend over WebSocket and also POSTs results directly.
- **Data model duality**: `Result` and `ScannedBib` each carry two nullable foreign keys — `race_id` (normal mode) and `simulationrun_id` (simulation mode) — exactly one is ever non-null, and queries must filter on the right one.
- **File locking**: a `threading.Lock` (`fileformat.filelock`) serializes any operation touching the CSV output file or related multi-table DB writes (place recalculation, scanned-bib queue assignment).
- **Race start time**: `Race.start_time` is the time-of-day offset added to Time Machine elapsed times when writing the CSV, and is shown on the results view for on-site verification. It can be set manually or auto-populated from the first live Trident `GUNTIME` marker for a race — but only once; later markers are ignored.
- **Release pipeline**: builds PyInstaller client executables, Sphinx docs, and a Docker image (via VS Code's "Build/Push/Release" task), then packages a main distribution zip plus a separate vendor-JS zip (only rebuilt when JS content actually changes, tracked via a committed SHA256 manifest).

## Gotchas worth knowing

- **Windows-only clients**: internal laptop Bluetooth (CNVi/PCIe-based) can't be passed through to a Docker container via USB/IP, so `tm-reader-client` must run on Windows directly — this is a deliberate, stable design choice, not a temporary workaround.
- **Local HTTPS via Caddy**: a separate `caddy-docker` reverse proxy provides automatic HTTPS for `*.localhost` domains in dev (`https://tm.localhost` → the app's nginx container). See [infrastructure/caddy.md](../infrastructure/caddy.md).
- **JS assets are shadowed by a Docker volume mount** to an external `js-common` directory (same shared pattern as scoretility/membertility/routetility) — editing the repo's own vendor-JS files directly has no effect on the running container.
- **Bind mounts only take effect on container recreation**, not a plain restart — `docker compose down && up` is needed if a mount is missing.
- **DataTables poll returns the full dataset every tick** (not incremental) — the results view can't use DataTables server-side mode because it needs to detect row deletions across the whole finisher list; composite indexes on `Result` keep the per-race query fast enough regardless.
- **Trident reader connectivity is a half-open-TCP trap**: a ping reply doesn't prove the telnet session to the reader is still alive. A reader that power-cycles without sending a FIN/RST leaves a dead socket that never reports an error. `trident-reader-client` therefore enables short-interval TCP keepalive, so the OS detects a half-dead socket and the client reconnects automatically. It also debounces status changes, and the results page shows a banner and beep for degraded connectivity. After an outage of a few minutes, the reader may refuse reconnects: each one is accepted and then closed at once, likely because it still holds the old session. If the chip reader keeps reconnecting while the network is up, power-cycle the reader.
- **Bluetooth barcode scanner dropouts are silent at the serial level**: when an SPP scanner powers off, Windows keeps its virtual COM port open and pyserial reports nothing. The scanner won't reconnect to a port that's still open; only closing and reopening the port restores the link. `barcode-scanner-client` polls the Windows Bluetooth link state, and when the link drops it closes the port and keeps reopening it until the scanner is back. While the scanner is off, each open fails with a `semaphore timeout`, and a scanner that isn't in Bluetooth SPP mode fails the same way. While a scanner or chip reader client is retrying, the results page shows it as *reconnecting* (orange button reading **Stop Reconnecting**, orange banner, beep) rather than as an ordinary disconnect, and the button cancels the retries.
- **Restart a client service with `Restart-Service <name>`, not `nssm restart`**: the clients don't exit on NSSM's Ctrl-C, so every stop takes ~3.4s while NSSM escalates to killing the process. nssm 2.24's `restart` doesn't wait for that. It reports `SERVICE_STOP_PENDING` and leaves the service stopped. `Restart-Service` waits, then starts it ([tm-csv-connector#153](https://github.com/louking/tm-csv-connector/issues/153)). A client *crash* is a different path: NSSM restarts the client by itself within ~2s, but it comes back disconnected from its device. The results page shows "client is not running" while the client is down, then a "client restarted -- click Connect" banner until the operator reconnects.
- **The Time Machine client is the least hardened**: unlike the scanner and chip reader clients, `tm-reader-client` has no open timeout, no guard against duplicate reader threads, no connecting/reconnecting status, and no reconnect loop. If the Time Machine doesn't respond, **Connect** hangs with no error or status on the page. Power-cycle the Time Machine and click **Connect** again ([tm-csv-connector#154](https://github.com/louking/tm-csv-connector/issues/154)).
- **A `None` race filter matches all simulation data**: results and scanned bibs belong to either a race or a simulation run, so filtering on a missing race (`race_id IS NULL`) returns every simulation-run row, not zero rows. On a first visit the results view's session has no race yet, so until the page sends its race (once the client WebSockets connect) the table briefly lists simulation results, and scan actions on those rows could touch simulation data. Code must check that a race or simulation run is present before filtering on it.
- **Clients keep the current race only in memory**: each client starts at race 0, and the database rejects anything it posts until it's told the real race. The browser sends the race on every race change, whenever a client's WebSocket (re)connects, and in every Connect (`open`) message. A race change always reaches the server even if a client is down; that client picks up the race when it reconnects.
- **Recovering cleared results**: Clear All saves a per-race snapshot of results and scans before deleting, and Undo Clear restores it. The Undo button only appears right after a clear, but the restore can also be called directly for a given race. Clearing the same race again overwrites its snapshot.

For full detail on any of the above (dev setup, full release pipeline, hardware protocols, JS architecture), see the repo's own `CLAUDE.md` linked above.
