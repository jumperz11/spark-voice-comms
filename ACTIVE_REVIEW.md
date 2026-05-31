# ACTIVE_REVIEW

Updated: 2026-05-31T16:39:11Z
Last automation run checked against: 2026-05-31T15:37:28.303Z

## Active JUMPERZ PRs (vibeforge1111/spark-voice-comms)

### PR #19
- URL: https://github.com/vibeforge1111/spark-voice-comms/pull/19
- Last verified state: open, mergeable=true, draft=false (from local tracker snapshot)
- Gate signal: `needs-security-redesign` flow still presumed active until live check succeeds
- Live check in this run: blocked (`gh` token invalid for `jumperz11`; `gh pr list` cannot reach `api.github.com`)

### PR #20
- URL: https://github.com/vibeforge1111/spark-voice-comms/pull/20
- Last verified state: open, mergeable=true, draft=false (from local tracker snapshot)
- Gate signal: `needs-security-redesign` flow still presumed active until live check succeeds
- Live check in this run: blocked (`gh` token invalid for `jumperz11`; `gh pr list` cannot reach `api.github.com`)

### PR #21
- URL: https://github.com/vibeforge1111/spark-voice-comms/pull/21
- Last verified state: open, mergeable=true, draft=false (from local tracker snapshot)
- Gate signal: `needs-security-redesign` flow still presumed active until live check succeeds
- Live check in this run: blocked (`gh` token invalid for `jumperz11`; `gh pr list` cannot reach `api.github.com`)

## New feedback since last run (2026-05-31T15:37:28.303Z)
- Unable to confirm due connectivity/auth blockers:
  - `gh auth status` reports invalid token for account `jumperz11`.
  - `gh pr list --repo vibeforge1111/spark-voice-comms ...` fails with `error connecting to api.github.com`.
  - `gh pr status --repo vibeforge1111/spark-voice-comms` fails with `error connecting to api.github.com`.

## Queue status
- No locally visible new maintainer/reviewer feedback artifacts.
- Live GitHub review queue verification is currently blocked.

## Spark/Telegram reproduction sweep (this run)
- Command evidence (local, 2026-05-31):
  - `/Users/jumperz/.spark/bin/spark status --json` -> `ok=false`, `spawner-ui` unhealthy/not Spark-supervised.
  - `/Users/jumperz/.spark/bin/spark verify --onboarding --json` -> `ok=false` (`module_health`, `builder_memory_direct_smoke`, `spawner_mission_relay`, `runtime_processes`).
  - `/Users/jumperz/.spark/bin/spark fix telegram --json` -> `ok=false` (`telegram_process`).
  - `/Users/jumperz/.spark/bin/spark restart telegram-starter` -> fails with `PermissionError: ... /Users/jumperz/.spark/state/pids.json.lock` in this sandbox.
- Ownership boundary decision: these failures are runtime/starter orchestration issues outside `spark-voice-comms` ownership, so no new voice-chip hotfix was opened from this reproduction.

## New hunt (distinct candidate)
- Candidate bug: `voice.speak` exposed raw hosted voice IDs in both `result.voice_id` and top-level `stdout`, even though runtime state already masks/fingerprints IDs.
- Distinctness scope used locally: no existing packet in this repo covers public `voice.speak` voice-id redaction for both result and stdout surfaces.

## Safe before/after proof
- Before (pre-fix behavior in test contract): speak responses returned raw values like `fake-elevenlabs-voice-id` / `coral` / `fallback-voice-id` in `result.voice_id`, and echoed raw IDs in `stdout`.
- After (current patch):
  - `result.voice_id` now returns masked values only.
  - `result.voice_id_masked` mirrors masked value.
  - `result.voice_id_fingerprint` returns short fingerprint.
  - `stdout` now mirrors masked IDs only (`provider:masked_voice_id`).

## Local verification run
- `python -m pytest -q tests/test_spark_hook.py -k "voice_speak"` -> 8 passed
- `python -m py_compile src/voice_comms_chip/spark_hook.py tests/test_spark_hook.py` -> passed
- `git diff --check` -> passed

## Scope in local diff
- `src/voice_comms_chip/spark_hook.py`: public `voice.speak` payload and top-level `stdout` now export masked/fingerprint-safe voice IDs instead of raw IDs
- `tests/test_spark_hook.py`: updated speak assertions to enforce masked/fingerprint-safe result contract and masked stdout output
- `docs/spark-compete-hotfix-voice-speak-voice-id-redaction.json`: new packet draft for this issue

## Blockers / next action
- Blocker: no live GitHub review/comment verification possible until auth/network path works in this sandbox.
- Blocker: local Spark onboarding is currently degraded by non-voice module/runtime checks (`spawner-ui`, Builder memory DB, mission relay), and restart attempts are blocked by local lockfile permissions in this sandbox.
- Next best action: once GitHub checks are reachable and runtime health is restored, verify no new reviewer instructions, then split this fix to a dedicated branch/PR with packet metadata aligned to that PR URL.
