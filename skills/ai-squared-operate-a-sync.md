---
name: ai-squared-operate-a-sync
description: >-
  Trigger, cancel, monitor and troubleshoot an AI Squared sync - fire a manual run, cancel it,
  read run-level tallies, and drill into individual failed rows. Use when an agent must operate or
  debug data movement that is already configured on the AI Squared platform.
api: AI Squared API
base_url: https://api.squared.ai/api/v1/
operations:
- manualSyncTrigger
- cancelSyncTrigger
- testSync
- listSyncs
- showSync
- updateSync
- deleteSync
- getSyncRuns
- getSyncRun
- getSyncRecords
generated: '2026-09-13'
method: generated
source: openapi/ai-squared-openapi.yml + conventions/ai-squared-conventions.yml
---

# Operate an AI Squared sync

All ten operations below carry provider-assigned operationIds from the published contract.

## Trigger a run

```
POST /api/v1/schedule_syncs              (manualSyncTrigger)
body: {"schedule_sync": {"sync_id": <integer>}}
```

Returns `200` with `{"message": "Sync scheduled successfully"}`.

## Cancel a run

```
DELETE /api/v1/schedule_syncs/{sync_id}  (cancelSyncTrigger)
```

This is the only reversal operation in the API. **No window is published** — the documentation
does not say whether an in-flight sync can be cancelled, nor whether rows already written to the
destination are rolled back. Do not tell a user their data movement has been undone; tell them the
sync was cancelled.

## Test instead of running

```
POST /enterprise/api/v1/syncs/{sync_id}/test   (testSync)
```

Enterprise deployments only. Prefer this to a live trigger when validating a change.

## Watch the run

```
GET /api/v1/syncs/{sync_id}/sync_runs                          (getSyncRuns)
GET /api/v1/syncs/{sync_id}/sync_runs/{sync_run_id}            (getSyncRun)
```

`getSyncRuns` accepts a `status` filter and `page[number]` / `page[size]` pagination. Each run
returns `started_at`, `finished_at`, `duration` and the tallies that matter:
`total_rows`, `successful_rows`, `failed_rows`, `skipped_rows`.

A run with `failed_rows > 0` is the signal to drill in. Both operations declare `404` "Sync not
found" — one of the few error responses in the contract.

## Drill into failures

```
GET /api/v1/syncs/{sync_id}/sync_runs/{sync_run_id}/sync_records   (getSyncRecords)
```

Returns the individual records with `status`, the `record` payload and any `error`. Filter with
`status` and page with `page[number]` / `page[size]`. Declares `404` "SyncRun not found".

This is the only row-level visibility the API offers. There is no aggregate error summary.

## Manage the sync itself

```
GET /api/v1/syncs             (listSyncs)
GET /api/v1/syncs/{id}        (showSync)
PUT /api/v1/syncs/{id}        (updateSync)
DELETE /api/v1/syncs/{id}     (deleteSync)
```

`updateSync` is a full replacement — send the complete object, not a patch. `deleteSync` returns
`204` and is permanent; there is no restore.

## Runtime rules

- 100 requests per minute across the whole API. Polling `getSyncRuns` in a tight loop will spend
  that budget; poll on an interval, not a spin.
- No `Retry-After` and no rate-limit headers. On a `429`, back off on your own schedule.
- No request-id or correlation header is returned, so when you escalate to AI Squared support you
  can cite the `sync_run_id`, not a trace id.
- Collection responses are `{"data": [...], "links": {"self","first","prev","next","last"}}`. There
  is no total count — follow `links.next` until it is absent rather than computing page counts.
