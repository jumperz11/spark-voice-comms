# ACTIVE_REVIEW

Updated: 2026-06-01T14:03:00Z
Last automation run checked against: 2026-06-01T11:51:57.497Z

## Active JUMPERZ PRs (vibeforge1111/spark-voice-comms)

### PR #19
- URL: https://github.com/vibeforge1111/spark-voice-comms/pull/19
- Metadata snapshot (`_get_pr_info`, this run): `state=open`, `merged=false`, `mergeable=true`, `draft=false`, `updated_at=2026-05-31T08:39:58Z`
- Labels/review gate signal (`_fetch_issue`, this run): `needs-security-redesign` still present; latest maintainer security-redesign comment remains at `2026-05-31T00:34:08Z`.
- Review state: no open inline review threads (`_list_pull_request_review_threads`, this run).
- Merge state: no merge/close transition detected.

### PR #20
- URL: https://github.com/vibeforge1111/spark-voice-comms/pull/20
- Metadata snapshot (`_get_pr_info`, this run): `state=open`, `merged=false`, `mergeable=true`, `draft=false`, `updated_at=2026-05-31T08:44:35Z`
- Labels/review gate signal (`_fetch_issue`, this run): `needs-security-redesign` still present; latest maintainer security-redesign comment remains at `2026-05-31T00:34:09Z`.
- Review state: no open inline review threads (`_list_pull_request_review_threads`, this run).
- Merge state: no merge/close transition detected.

### PR #21
- URL: https://github.com/vibeforge1111/spark-voice-comms/pull/21
- Metadata snapshot (`_get_pr_info`, this run): `state=open`, `merged=false`, `mergeable=true`, `draft=false`, `updated_at=2026-05-31T08:50:13Z`
- Labels/review gate signal (`_fetch_issue`, this run): `needs-security-redesign` still present; latest maintainer security-redesign comment remains at `2026-05-31T00:34:09Z`.
- Review state: no open inline review threads (`_list_pull_request_review_threads`, this run).
- Merge state: no merge/close transition detected.

## New feedback since last run (2026-06-01T11:51:57.497Z)
- GitHub connector checks in this run:
  - `_get_pr_info` + `_fetch_pr_comments` + `_fetch_issue` for PRs #19/#20/#21.
  - `_list_pull_request_review_threads` and `_list_pull_request_reviews` for PRs #19/#20/#21.
- Outcome: no maintainer/reviewer comments newer than `2026-05-31T08:50:13Z`; no new review submissions; no new inline review threads.

## Queue status
- No newly verified actionable reviewer tasks.
- Known gate unchanged: security-owner review hold (`needs-security-redesign`) before Mac Lab across PRs #19/#20/#21.

## Spark/Telegram reproduction sweep (this run)
- Local command evidence (`2026-06-01T14:00Z`):
  - `/Users/jumperz/.spark/bin/spark status --json` -> `ok=false`; `spark-voice-comms` module healthy, but runtime degraded by `spawner-ui` unhealthy/not Spark-trusted.
  - `/Users/jumperz/.spark/bin/spark verify --onboarding --json` -> `ok=false` with required failures: `module_health` (`spawner-ui`), `builder_memory_direct_smoke` (`sqlite3.OperationalError: attempt to write a readonly database`), `spawner_mission_relay`, `runtime_processes`.
  - `/Users/jumperz/.spark/bin/spark fix telegram --json` -> `ok=false`; `telegram_process` still reports `spark-telegram-bot is not running under Spark supervision`.
- Ownership boundary decision: these remain Spark runtime/orchestration gates outside `spark-voice-comms` ownership.

## New hunt status
- Distinct voice-owned bug reproduced and fixed locally in this run:
  - Bug: `voice.transcribe` deterministic fallback smoke (`examples/voice_transcribe_fallback.json`) failed when local `faster_whisper` was unavailable, despite `fallback_mode=deterministic`.
  - Before proof: CLI returned `returncode=1` with `Local faster-whisper transcription is the default ... not installed`.
  - After proof: same CLI call now returns `returncode=0` with `mode=deterministic_fallback` and bounded fallback reason.
- Local code changes:
  - `src/voice_comms_chip/spark_hook.py`: allow deterministic fallback in the local-missing STT branch.
  - `tests/test_spark_hook.py`: add regression test `test_voice_transcribe_deterministic_fallback_when_local_stt_is_unavailable`.
  - `docs/spark-compete-hotfix-voice-transcribe-deterministic-fallback-local-missing.json`: packet draft for this issue.
- Verification in this run:
  - `python -m pytest -q tests/test_spark_hook.py -k 'deterministic_fallback or voice_transcribe_prefers_local_faster_whisper_without_openai_call_when_available or voice_transcribe_uses_local_by_default_and_does_not_silently_fallback_to_hosted_when_local_missing'` -> `4 passed`.
  - `PYTHONPATH=src python -m voice_comms_chip.spark_hook voice.transcribe --input examples/voice_transcribe_fallback.json --output <tmp>` -> `rc=0`, `mode=deterministic_fallback`.
  - `python -m py_compile src/voice_comms_chip/spark_hook.py tests/test_spark_hook.py` -> passed.
  - `git diff --check` -> passed.

## Blockers / next action
- Blocker: PR review queue still gated by security-owner hold (`needs-security-redesign`) with no fresh maintainer feedback.
- Blocker: local Spark onboarding remains degraded due non-voice runtime checks (`spawner-ui`, readonly Builder DB, mission relay).
- Next best action: publish the new deterministic-fallback local-missing fix as a focused Spark Compete PR with the packet above, then continue polling PR #19/#20/#21 for fresh maintainer/security-owner comments.
