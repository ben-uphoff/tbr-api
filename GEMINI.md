# The Bot Range (TBR) - Agent Rules (Gemini / Antigravity)

## 1. Source of Truth
- Authoritative API contract: `openapi.yaml`. Inspect endpoint request/response schemas before writing HTTP calls.
- Game mechanics & formulas: `FIELD_MANUAL.md`.

## 2. HTTP Loop Guardrails
- **Synchronous & Atomic**: Run 1 action per bot tick sequentially (`GET /game/state` -> decision -> action). Do not fire concurrent mutating calls.
- **HTTP 429 (`rate_limited`)**: Extract `seconds_to_wait` and sleep. Never hammer a rate-limited endpoint.
- **HTTP 502 (`signal_interference`)**: Zero tokens lost. Apply a 150-400ms jittered backoff and retry.
- **Zone Boundaries**: Always include `"zone_change": true` when issuing `POST /game/move`.

## 3. Movement & Extraction Economics
- Moving towards the center extraction zone (`d_new < min_extraction_distance`) is a **Free Move** (0 action tokens consumed).
- Moving laterally, outward, or within Ring 0 costs 1 token. Do not oscillate between tiles (anti-ping-pong penalty).

## 4. Loadout & Run Lifecycle
- `POST /game/loadout` and `POST /game/fabricate` are valid ONLY when extracted/offline. Loadouts cannot be swapped mid-run.
- On bot death/despawn, inspect respawn cooldown from `GET /game/state` before calling `POST /game/spawn`.

## 5. Mission 01 Objective
- Complete a full autonomous cycle: spawn -> explore -> defeat at least 1 hostile -> extract safely at Ring 0 with `< 50%` HP fallback retreat.
