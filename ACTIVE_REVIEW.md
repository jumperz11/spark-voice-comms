# ACTIVE_REVIEW

Updated: 2026-05-31T21:56:00Z
Last automation run checked against: 2026-05-31T20:43:49.649Z

## Active JUMPERZ PRs (vibeforge1111/spark-voice-comms)

### PR #19
- URL: https://github.com/vibeforge1111/spark-voice-comms/pull/19
- Last verified state: open, mergeable=true, draft=false, updated_at=2026-05-31T08:39:58Z (GitHub connector live check)
- Labels: `needs-security-redesign`
- Review state: no open inline review threads
- Merge state: open and mergeable; not merged

### PR #20
- URL: https://github.com/vibeforge1111/spark-voice-comms/pull/20
- Last verified state: open, mergeable=true, draft=false, updated_at=2026-05-31T08:44:35Z (GitHub connector live check)
- Labels: `needs-security-redesign`
- Review state: no open inline review threads
- Merge state: open and mergeable; not merged

### PR #21
- URL: https://github.com/vibeforge1111/spark-voice-comms/pull/21
- Last verified state: open, mergeable=true, draft=false, updated_at=2026-05-31T08:50:13Z (GitHub connector live check)
- Labels: `needs-security-redesign`
- Review state: no open inline review threads
- Merge state: open and mergeable; not merged

## New feedback since last run (2026-05-31T20:43:49.649Z)
- No new maintainer/reviewer comments detected on PRs #19/#20/#21 after the last-run timestamp.
- Latest maintainer guidance remains the known security-owner review gate before Mac Lab.
- Local `gh` CLI is still blocked by network (`error connecting to api.github.com`), but connector checks succeeded for PR metadata/comments/review threads.

## Queue status
- No new actionable reviewer tasks in this run.
- Existing gate remains: security-owner review before Mac Lab on PRs #19/#20/#21.

## Spark/Telegram reproduction sweep (this run)
- Command evidence (local, 2026-05-31T21:56:00Z):
  - `/Users/jumperz/.spark/bin/spark status --json` -> `ok=false`; `spark-voice-comms` module is healthy, but `spawner-ui` is unhealthy (not Spark-supervised) and telegram process supervision is blocked by dependency health.
  - `/Users/jumperz/.spark/bin/spark verify --onboarding --json` -> `ok=false` with failed required checks: `module_health` (`spawner-ui`), `builder_memory_direct_smoke` (`sqlite3.OperationalError: attempt to write a readonly database`), `spawner_mission_relay`, `runtime_processes`.
  - `/Users/jumperz/.spark/bin/spark fix telegram --json` -> `ok=false` with `telegram_process` failed (`spark-telegram-bot` not running under Spark supervision).
- Ownership boundary decision: these failures are runtime/orchestration checks outside `spark-voice-comms` ownership, so no new voice-chip hotfix was opened from this reproduction.

## New hunt (distinct candidate)
- Candidate bug: `voice.speak` exposed raw hosted voice IDs in both `result.voice_id` and top-level `stdout`, even though runtime state already masks/fingerprints IDs.
- Distinctness scope used locally: no current open PR in this repo covers `voice.speak` public voice-id redaction for both result and stdout surfaces.

## Safe before/after proof
- Before (pre-fix contract): speak responses returned raw values like `fake-elevenlabs-voice-id` / `coral` / `fallback-voice-id` in `result.voice_id`, and echoed raw IDs in `stdout`.
- After (current patch):
  - `result.voice_id` now returns masked values only.
  - `result.voice_id_masked` mirrors masked value.
  - `result.voice_id_fingerprint` returns short fingerprint.
  - `stdout` now mirrors masked IDs only (`provider:masked_voice_id`).

## Local verification run
- `python -m pytest -q tests/test_spark_hook.py -k "voice_speak"` -> 8 passed, 37 deselected
- `python -m py_compile src/voice_comms_chip/spark_hook.py tests/test_spark_hook.py` -> passed
- `git diff --check` -> passed

## Scope in local diff
- `src/voice_comms_chip/spark_hook.py`: public `voice.speak` payload and top-level `stdout` now export masked/fingerprint-safe voice IDs instead of raw IDs
- `tests/test_spark_hook.py`: updated speak assertions to enforce masked/fingerprint-safe result contract and masked stdout output
- `docs/spark-compete-hotfix-voice-speak-voice-id-redaction.json`: packet draft for this issue

## Blockers / next action
- Blocker: existing open PRs are still waiting on maintainer/security-owner review gates before Mac Lab.
- Blocker: local Spark onboarding remains degraded by non-voice runtime/starter checks (`spawner-ui`, Builder memory DB, mission relay), so a full Telegram-green proof cannot be claimed from this sandbox.
- Next best action: split the `voice.speak` voice-id redaction fix to a dedicated branch/PR with packet target URL set to that PR, then post concise safe before/after proof plus focused test output.
