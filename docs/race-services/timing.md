# Timing

Covers how a race gets timed on race day: the **Trident** chip timing system as primary source, the **Time Machine** manual timer (feeding [tmtility](../applications/tmtility.md)) as backup, and finish-line video as a secondary backup for resolving disputes. Output from both streams feeds **RaceDay Scoring**.

- Sources: [Chip Timing Configuration and Operation](https://docs.google.com/document/d/1utAcupx2zqPyF6dONEBtDsQsB651qulztP_W2txLqsc/edit?usp=sharing) (theory/operation) and [Chip/Scoring RaceDay Scoring Checklist](https://docs.google.com/spreadsheets/d/1GcDuB508jKu_kdMXZ8VhzIaZk6LE2l4qIxwpstLUKIc/edit?usp=sharing) (per-race setup steps) — authoritative, deeper detail than this page

## Chip vs. scoring-only races

FSRC runs races in one of two RaceDay Scoring configurations, depending on whether Trident chip timing is deployed for that race:

- **Chip** races: Trident is the main stream, with Time Machine and a Trident-file backup stream, a single `Start/Finish` timing location, and results sorted by Chip Time.
- **Scoring-only** races: no Trident. Time Machine is the only stream, assigned to a `Finish`-only timing location (there's no timed start to capture), Actual Start Time is set manually, and there's a single finish occurrence.

The rest of this page marks configuration values as **(chip)** or **(scoring)** where they differ between the two.

## Theory of operation

### Last Seen, First Seen, and Best Seen

Trident's UHF8 reader transmits tag ID reads roughly 100 times per second, but only forwards two per tag to scoring software: **Last Seen (LS)** and **Best Seen (BS)**. Per Trident's docs, the BS timestamp is always before or at the LS timestamp.

RaceDay Scoring (RDS) uses LS at the start line and BS at the finish. Body position relative to the antenna mat means a runner's tag is read repeatedly as they cross — LS captures the true crossing moment at the start, while at the finish, Trident's own algorithm (Best Seen) compensates for early reads and takes about half a second to settle, per Trident support.

### Start/finish determination

Requires setting the race's Actual Start Time to the first `GUNTIME` marker, plus time windows for start/finish read collection and Max Occurrences / Gap Factor settings (see the [RaceDay Scoring setup checklist](#raceday-scoring-per-race-setup-checklist) below).

### Bib/chip association

Chip IDs are linked to bib numbers via CSV upload; up to two chips per bib are supported. Trident chip IDs are the number printed on the chip, zero-filled to 12 digits.

### Missed finish reads

Time Machine (TM) serves as the finish-line backup stream. RDS ignores backup-stream reads entirely once any main-stream (Trident) read exists for a participant — "as soon as a read is seen on a main stream for someone we will immediately ignore any times from any backup streams" (RDS docs). When TM reads are actually needed, a Read Cutoff Time should be set to stop later errant chip reads from overriding the real finish time.

## Configuration reference

### Trident (`config.ini`)

- Time zone offset: `-04:00` (EDT) or `-05:00` (EST) — re-verify per race day, not just once per season, since EDT/EST changes across it
- System name: `FSRC_UHF8_A`
- Filtered message types: `BS,LS`
- GPS autocorrect limit: `-1`

### RaceDay Scoring: per-race setup checklist

Run through the [Chip/Scoring RaceDay Scoring Checklist](https://docs.google.com/spreadsheets/d/1GcDuB508jKu_kdMXZ8VhzIaZk6LE2l4qIxwpstLUKIc/edit?usp=sharing) for every race, whether **(chip)** or **(scoring)**-only (see [above](#chip-vs-scoring-only-races)) — it's the authoritative, field-by-field walkthrough (RunSignup import → Scored Events → Timing Locations → Streams → Sync/participants → Age Groups & Awards → Data Checks → Reports → Wrap-up), link-shareable to anyone with the URL.

A few stable configuration values worth having on hand without opening the sheet:
- Trident stream (**chip only**): Direct Connect, reader `FSRC_UHF8_A`, IP `192.168.0.101` port `10001`.
- Time Machine stream: File type, folder `C:\Users\fsrck\Documents\tm-data`, extension `csv`, passing format `[IGNORE],[BIBCODE],[TIME]`, comma-delimited.
- Trident File backup stream (**chip only**, alongside Time Machine): File type, folder `C:\Users\fsrck\Documents\trident-data`, extension `log`.
- Min Elapsed Finish Time Allowed / Gap Factor scale by distance: 14 min (5K), 30 min (10K), 50 min (10M).

### tmtility

Race Start Time should be set immediately after the race begins, to the first `GUNTIME` marker (see [tmtility](../applications/tmtility.md) and its [user's guide](https://tm-csv-connector.readthedocs.io/en/latest/users-guide.html#tmtility-operation) — this can also auto-populate from the live Trident marker, once, on first arrival, and is shown on the results view so it can be checked on-site).

### Finish-line video backup

- **Camera**: Topodome TD-J10A bullet camera, fixed IP `192.168.0.102`, timestamp overlay (bottom-left) enabled, 15fps, audio disabled, NTP/timezone configured, automatic DST adjustment disabled.
- **Recording laptop**: external hard drive mounted as `D:\`, sleep disabled while plugged in.
- **Blue Iris** (DVR software): new-clip folder `D:\BlueIris\New` (1000 GB), stored-clip folder `D:\BlueIris\Stored` (0 GB, i.e. unused/disabled), clip age limits disabled; live preview capped at 5fps with overlays off; continuous recording with 1-hour/3.0 GB file segmentation and direct-to-disk compression to keep CPU usage down.

## Race-day operation

**Before start**: sync laptop time; open the IP camera (`192.168.0.102`) and check/adjust its time.

**At start**: pull the gun trigger (records `GUNTIME`); start the display clock; press the Time Machine's START TIME button.

**After start**: set the race's Start Time in tmtility to match `GUNTIME`, using the Raw Reads filter to locate the first `GUNTIME` marker if it wasn't auto-populated.

**During the race**: watch results/confirmations in tmtility (including the Start Time shown on its results view) and recent reads on the RDS Dashboard; check Time Machine raw reads for any missed finish reads and set a Read Cutoff Time if needed.

**Post-race**: run through the [RaceDay Scoring setup checklist's](#raceday-scoring-per-race-setup-checklist) Data Checks/Reports/Wrap-up steps; upload Trident's `.log` files via WinSCP to `\\fsrc-race-serv2\users\Public\past races\Trident Logs` then delete them from the reader; see RunSignup's [Adding Video Results to Your Race](https://help.runsignup.com/support/solutions/articles/17000073009-adding-video-results-to-your-race) for pulling video results into the results workflow.

## Naming gotcha

Both Trident and Flying Feet Computers use "TimeMachine" as a product name. In this wiki (and in tmtility), **Trident** refers to the UHF8 chip-timing system, and **Time Machine (TM)** refers to the separate manual backup timer at timemachine.org — see the [glossary](../GLOSSARY.md).

## References

- [Trident TimeMachine Manual](https://www.manula.com/manuals/tridentrfid/timemachine/1/en/topic/getting-started) and its [Configuration Options](https://www.manula.com/manuals/tridentrfid/timemachine/1/en/topic/configuration-options) page
- [RaceDay Scoring support solutions](https://help.rdscoring.com/support/solutions)
- [Optimizing Blue Iris's CPU Usage](https://ipcamtalk.com/wiki/optimizing-blue-iris-s-cpu-usage/) and [Blue Iris Clip Length For Continuous Recording](https://ipcamtalk.com/threads/blue-iris-clip-length-for-continuous-recording.61071/post-630669) (IP Cam Talk forum)
- [tmtility user's guide: tmtility Operation](https://tm-csv-connector.readthedocs.io/en/latest/users-guide.html#tmtility-operation)
- [RunSignup: Adding Video Results to Your Race](https://help.runsignup.com/support/solutions/articles/17000073009-adding-video-results-to-your-race)
