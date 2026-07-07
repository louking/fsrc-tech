# Glossary

FSRC- and running-club-specific terms used throughout this wiki. Generic software terms (Flask, Docker, etc.) are linked inline where they appear instead of defined here.

- **Equalizer** — Another scored series, similar in structure to the Grand Prix, handicapping/age-grading results so runners of different abilities compete fairly. Scored by scoretility.
- **FSRC** — Frederick Steeplechasers Running Club, the club all of these apps serve.
- **FSRC Community** (also "FSRC Community forum") — The club's online forum, managed by membertility's community module (group membership, event topics, calendar feeds). Built on the open-source [Discourse](https://www.discourse.org/) platform, but referred to by its FSRC-facing name rather than the underlying framework — parallel to how steeplechasers.org isn't usually called "WordPress". Use "Discourse" only when the underlying framework/API is actually what's being discussed.
- **Grand Prix** — FSRC's points-based race series; scored by [scoretility](applications/scoretility.md).
- **interest** — The multi-tenancy concept used in [membertility](applications/membertility.md): identifies which club/program a request belongs to (e.g., different series or even different clubs sharing the same app instance). Appears in URLs as `/app/<interest>/...` and in config keys as a suffix (e.g., `DISCOURSE_API_URL_<INTEREST>`).
- **MailChimp** — Email list service used for club communications; synced from membertility's membership module.
- **Maryland/DC Grand Prix** — A separate multi-club Grand Prix series (not FSRC-only) also scored by scoretility.
- **RaceDay Scoring** — Third-party race-day scoring software. tmtility produces CSV output formatted for it.
- **Racing Team** — FSRC's competitive team; membertility's racing team module handles applications, race-result submission, and volunteer tracking for its members.
- **RunSignup** — Third-party race registration and club membership platform. FSRC's membership data is synced from RunSignup into membertility; race results/participants also feed the awards and community modules.
- **Time Machine** — A manual race-timer hardware/service (timemachine.org) whose output tmtility ingests.
- **xtility** — Umbrella name (used internally in `loutilities/user/roles.py`) for the app suite sharing the common [loutilities](framework/loutilities.md)-based user/role/SSO layer — the source of the "-tility" suffix in scoretility, routetility, contractility, membertility, tmtility.
