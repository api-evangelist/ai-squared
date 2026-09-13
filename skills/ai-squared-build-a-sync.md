---
name: ai-squared-build-a-sync
description: >-
  Stand up a complete AI Squared data-activation pipeline over the REST API - connect a source and
  a destination, register the destination catalog, define the model that selects the data, and
  create the sync that moves it. Use when an agent must wire a warehouse to a business
  application on the AI Squared platform.
api: AI Squared API
base_url: https://api.squared.ai/api/v1/
operations:
- GET /api/v1/connector_definitions
- GET /api/v1/connector_definitions/{connector_name}
- POST /api/v1/connector_definitions/check_connection
- POST /api/v1/connectors
- GET /api/v1/connectors/{id}/discover
- POST /api/v1/catalogs
- POST /api/v1/models
- POST /api/v1/syncs
- createCatalog
- createSync
generated: '2026-09-13'
method: generated
source: openapi/ai-squared-openapi.yml + conventions/ai-squared-conventions.yml
---

# Build an AI Squared sync

Every operation below is in the published contract (`openapi/ai-squared-openapi.yml`). Two of them
carry provider-assigned operationIds (`createCatalog`, `createSync`); the rest are addressed by
method and path because the provider did not assign them ids.

## Before you start

- **Auth.** Every call needs `Authorization: Bearer <JWT>`. The token is generated in the AI
  Squared dashboard. There is no OAuth flow and no API-key scheme.
- **Base URL.** `https://api.squared.ai/api/v1/`. The one exception in this API is the sync test
  operation, which lives under `/enterprise/api/v1/`.
- **Rate limit.** 100 requests per minute, `429` on exhaustion. No `RateLimit-*` or `Retry-After`
  header is returned, so pace yourself rather than waiting to be told.
- **No idempotency.** There is no `Idempotency-Key` header on this API. If a `POST` times out, do
  **not** blind-retry it — list the collection first (`GET /api/v1/connectors`,
  `GET /api/v1/syncs`) and check whether the object was created.

## Step 1 — find the connector type

```
GET /api/v1/connector_definitions
GET /api/v1/connector_definitions/{connector_name}
```

The definition returns `connection_spec` (what configuration this connector needs),
`release_stage`, `support_level` and `documentation_url`. Read `connection_spec` before building
any configuration object — it is the only published description of the required fields.

## Step 2 — rehearse the connection

```
POST /api/v1/connector_definitions/check_connection
```

This validates a configuration **without creating anything**. Run it for both the source and the
destination before Step 3. It is the only pre-flight this API offers.

## Step 3 — create the source and the destination

```
POST /api/v1/connectors
```

Send `connector_type` (`source` or `destination`), `connector_name`, `connector_subtype`, a
`name`, and the `configuration` shaped by `connection_spec`. Returns `201` with the connector
`id`. Do this twice — once for the source, once for the destination.

Reversal: `DELETE /api/v1/connectors/{id}`. Deletion is permanent; there is no restore.

## Step 4 — discover the schema

```
GET /api/v1/connectors/{id}/discover
```

Returns the connector's catalog — the `streams[]` it can read or write, each with a `json_schema`.
Pass `refresh=true` to force a re-read rather than a cached one.

## Step 5 — register the destination catalog

```
POST /api/v1/catalogs        (operationId: createCatalog)
```

Send `workspace_id`, `connector_id` and a `catalog` object whose `streams[]` entries carry `name`,
`url`, `json_schema`, `request_method`, `batch_support` and `batch_size`. Set the outbound
throttles here — `request_rate_limit`, `request_rate_limit_unit`, `request_rate_concurrency` —
which govern how hard AI Squared writes into the destination's own API.

This operation is one of only two in the whole contract that declares error responses: `400`,
`401`, `404`, `500`. There is no `GET` and no `DELETE` for catalogs — a mistake here is corrected
with `PUT /api/v1/catalogs/{id}` (`updateCatalog`), never undone.

## Step 6 — define the model

```
POST /api/v1/models
```

Send `connector_id`, `name`, `query`, `query_type` and `primary_key`. The model is what selects
rows out of the source; the `primary_key` is what makes incremental syncs possible.

## Step 7 — create the sync

```
POST /api/v1/syncs           (operationId: createSync)
```

Send `source_id`, `model_id`, `destination_id`, `stream_name`, `sync_mode`, `cursor_field` and a
`configuration` field mapping. Schedule with `schedule_type` plus either `cron_expression` or
`sync_interval` / `sync_interval_unit`.

Check `GET /api/v1/syncs/configurations` (`getConfigurations`) first — it returns the
`catalog_mapping_types` and transformation primitives (`standard`, `static`, `template`, `cast`,
`regex_replace`, `current_timestamp`, `variable`) that the `configuration` object accepts.

## Step 8 — verify before you let it run

On enterprise deployments, `POST /enterprise/api/v1/syncs/{sync_id}/test` (`testSync`) triggers a
test of the configured sync. If that path is not available to you, trigger one manual run and
inspect it rather than waiting for the schedule — see `ai-squared-operate-a-sync`.

## What you cannot undo

A sync writes customer data into a live destination. Nothing in this API reverses rows already
delivered. `DELETE /api/v1/syncs/{id}` and `DELETE /api/v1/schedule_syncs/{sync_id}` stop future
movement only. Treat Step 8 as mandatory, not optional.
