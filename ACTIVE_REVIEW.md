# ACTIVE_REVIEW

Updated: 2026-05-31T10:27:45Z
Last automation run checked against: 2026-05-31T09:22:20.495Z

## Active JUMPERZ PRs (vibeforge1111/spark-voice-comms)

### PR #19
- URL: https://github.com/vibeforge1111/spark-voice-comms/pull/19
- State: open, mergeable=true, draft=false
- Gate signal: still under `needs-security-redesign` flow via maintainer comments
- Latest maintainer comment: 2026-05-31T00:34:08Z (`spark-compete-feedback:v1` security-first redesign reminder)
- Latest JUMPERZ reply: 2026-05-31T08:39:57Z (branch reduced to focused unsupported-runtime fix, safe proof rerun)
- Review threads: none
- Review submissions: none

### PR #20
- URL: https://github.com/vibeforge1111/spark-voice-comms/pull/20
- State: open, mergeable=true, draft=false
- Gate signal: still under `needs-security-redesign` flow via maintainer comments
- Latest maintainer comment: 2026-05-31T00:34:09Z (`spark-compete-feedback:v1` security-first redesign reminder)
- Latest JUMPERZ reply: 2026-05-31T08:44:35Z (reduced to hook input/error-envelope root cause, safe proof rerun)
- Review threads: none
- Review submissions: none

### PR #21
- URL: https://github.com/vibeforge1111/spark-voice-comms/pull/21
- State: open, mergeable=true, draft=false
- Gate signal: still under `needs-security-redesign` flow via maintainer comments
- Latest maintainer comment: 2026-05-31T00:34:09Z (`spark-compete-feedback:v1` security-first redesign reminder)
- Latest JUMPERZ reply: 2026-05-31T08:50:13Z (reduced to transcribe audio boundary root cause, safe proof rerun)
- Review threads: none
- Review submissions: none

## New feedback since last run (2026-05-31T09:22:20.495Z)
- None detected on PRs #19, #20, #21.

## Queue status
- Maintainer/security feedback queue is currently clean (no newer comments than last run).
- All three active PRs remain blocked on security-owner review gates.

## New hunt (distinct candidate)
- Candidate bug: `voice.speak` can echo raw `telegram_delivery.failure_reason` text into `result.delivery_trace.failure_reason` and `runtime_state.telegram_delivery.last_failure_reason`.
- Distinctness check:
  - No open PR hit for `telegram_delivery` / `failure_reason` / delivery-trace redaction in upstream search.
  - Removed local duplicate `.env` parser changes because open PR #22 already covers quote/export parsing.

## Safe before/after proof
- Before (`origin/main` at `75277a7`): reproduction shows fake key and temp env-file path echoed in delivery/runtime failure reason fields.
- After (local patch): same reproduction returns redacted reason text; key/path no longer present.

## Local verification run
- `python -m pytest -q tests/test_spark_hook.py -k "voice_speak_redacts_telegram_delivery_failure_reason or voice_speak_uses_telegram_compatible_opus_for_telegram_surface"` -> 2 passed
- `python -m pytest -q tests/test_spark_hook.py` -> 45 passed
- `python -m py_compile src/voice_comms_chip/spark_hook.py tests/test_spark_hook.py` -> passed
- `git diff --check` -> passed

## Scope in local diff
- `src/voice_comms_chip/spark_hook.py`: sanitize `telegram_delivery` failure reason before runtime-state/delivery-trace export
- `tests/test_spark_hook.py`: add focused regression for delivery-reason redaction
- Draft packet artifact retained: `docs/spark-compete-hotfix-voice-speak-delivery-reason-redaction.json` (needs final PR URL/branch alignment before submission)

## Blockers / next action
- Blocker: no new maintainer prompts to answer on current PRs; existing PRs remain in security-review holding pattern.
- Next best action: split this candidate onto a dedicated branch/PR (`voice.speak` redaction only) with refreshed packet metadata and public-safe proof excerpt.
