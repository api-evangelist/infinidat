---
name: Onboard a host onto an InfiniBox array
description: Register an initiator host on an InfiniBox array, confirm the array is healthy and has a pool with capacity, and tag the new host with ownership metadata so the inventory stays attributable.
api: openapi/infinidat-infinibox-openapi.yml
operations:
  - login
  - systemGetReadyStatus
  - systemGetHealthState
  - poolGetPools
  - hostCreate
  - metatdataAddToStorageEntity
  - metatdataGetByObject
  - logout
generated: '2026-08-01'
method: generated
source: openapi/infinidat-infinibox-openapi.yml
---

# Onboard a host onto an InfiniBox array

This is a **write** flow. Read the cautions before you run it.

## Preconditions

1. **Authenticate.** `login` — `POST /users/login`.
2. **Confirm the array is ready.** `systemGetReadyStatus` — `GET /system/ready`, and
   `systemGetHealthState` — `GET /system/health_state`. Do not provision onto an array that is
   reporting a degraded health state without a human decision.
3. **Pick the pool.** `poolGetPools` — `GET /pools?name=eq:<pool-name>`, or `GET /pools?id=<id>`.
   Confirm it exists and has room before you create anything.

## Create the host

4. **Register the initiator host.** `hostCreate` — `POST /hosts`.

   **There is no idempotency key on this API.** If the request times out you do not know whether
   the host was created. Do not blindly retry: re-read the host list first and only re-issue if
   the host genuinely is not there. This is the single most important operational fact about
   writing to InfiniBox.

5. **Handle the approval gate.** If the response carries an `error.code` of `APPROVAL_REQUIRED`
   or `APPROVAL_REQUIRED_VOLUME_HAS_CHILDREN`, read `error.reasons[0]`, surface it to a human,
   and only re-issue the identical request with `?approved=true` once approved.

## Make it attributable

6. **Tag the host.** `metatdataAddToStorageEntity` — `PUT /metadata/{object_id}` with the new
   host's id as `object_id`. Write the owning team, application, cost centre and change ticket.
   InfiniBox's metadata surface is free-form key/value, and it is the only thing that will let a
   future inventory answer "who owns this".
7. **Verify the tag landed.** `metatdataGetByObject` — `GET /metadata/{object_id}`.
8. **Log out.** `logout` — `POST /users/logout`.

## What this skill deliberately does not do

Mapping volumes to the host (assigning LUNs), creating the volumes themselves, and creating
clusters are **not** covered, because those paths are not present in the API description derived
from Infinidat's published 7.3 collection. Rather than guess a path, use one of Infinidat's own
first-party tools for those steps:

- `infinidat.infinibox` Ansible collection — `infini_vol`, `infini_map`, `infini_cluster`,
  `infini_host`, `infini_port`
- InfiniSDK — `system.volumes`, `system.hosts`, `system.host_clusters`
- InfiniShell

See `data-model/infinidat-data-model.yml` for which entities are confirmed in the spec and which
are documented elsewhere.

## Errors

- **401** — session expired; re-login once and retry.
- **403 / `REMOTE_PERMISSION_REQUIRED`** — you reached a replication peer; supply its credentials
  on `X-Remote-Authorization`.
- **503** — array unavailable; back off. Because there is no idempotency key, re-check state
  before re-issuing any write that may have partially applied.
