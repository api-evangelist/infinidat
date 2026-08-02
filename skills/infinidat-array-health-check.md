---
name: Check the health of an InfiniBox array
description: Authenticate against an InfiniBox array and read its name, readiness, health state, capacity and live statistics, then walk the hardware component tree to find any node, port or drive that is not OK.
api: openapi/infinidat-infinibox-openapi.yml
operations:
  - login
  - systemGetName
  - systemGetReadyStatus
  - systemGetHealthState
  - systemGetTotalVirtualCapacity
  - systemGetStatistics
  - componentsGet
  - nodeGetNodes
  - nodeGetNode
  - logout
generated: '2026-08-01'
method: generated
source: openapi/infinidat-infinibox-openapi.yml
---

# Check the health of an InfiniBox array

Read-only. Every operation in this skill is a GET except the login and logout that bracket it.

## Before you start

- The base URL is the customer's own array: `https://{infinibox_host}/api/rest`. There is no
  public Infinidat endpoint and no sandbox — you must be given a hostname and credentials.
- Every response is the envelope `{ "result": ..., "metadata": {...}, "error": null }`. Read your
  data from `result`, never from the top level.
- Read the `X-INFINIDAT-VERSION` response header and record it. It is how the array tells you
  which API version you are talking to; this skill was written against 7.3.
- If any response carries an `x-infinidat-deprecated-api` header, surface it — the array is
  telling you the call you just made is going away.

## Steps

1. **Authenticate.** `login` — `POST /users/login` with `{"username": ..., "password": ...}`.
   This sets a session cookie. HTTP Basic also works if you prefer stateless calls.
2. **Identify the array.** `systemGetName` — `GET /system/name`. Log the name alongside every
   finding so a multi-array report is unambiguous.
3. **Check readiness.** `systemGetReadyStatus` — `GET /system/ready`. If the array is not ready,
   stop and report that; the numbers below will be misleading.
4. **Check health.** `systemGetHealthState` — `GET /system/health_state`. This is the headline.
5. **Check capacity.** `systemGetTotalVirtualCapacity` — `GET /system/capacity/total_virtual_capacity`.
6. **Check load.** `systemGetStatistics` — `GET /system/stats`.
7. **Walk the hardware.** `componentsGet` — `GET /components` for the whole tree, or go narrow:
   `nodeGetNodes` — `GET /components/racks/{rack_id}/nodes` and then `nodeGetNode` —
   `GET /components/racks/{rack_id}/nodes/{node_id}` for one controller.
8. **Triage the component states.** Each node carries `state` and `state_description`, plus
   nested `ipmi.state`, `ntp.state` and `bios.state`, and arrays of `fc_ports` and `drives` that
   each carry their own `state`. Report everything that is not `OK`, quoting `state_description`
   verbatim — for example a drive can be `WARNING` with
   `"drive is not certified (not in whitelist)"`, which is a supply-chain finding, not a failure.
9. **Log out.** `logout` — `POST /users/logout`.

## Making the reads cheap

The component tree is large. Use the documented query conventions on any GET:

- `?fields=id,name,state,state_description` to cut the payload down to what you are triaging.
- `?page=1&page_size=1000` — default page size is 50, maximum is 1000. Keep paging while
  `metadata.page < metadata.pages_total`.
- `?state=ne:OK` to ask the array for the exceptions instead of filtering client-side.
  Operators available: `eq:`, `ne:`, `like:`, range comparisons, and membership.
- `?sort=id`, or `?sort=-id` to reverse.

## Errors you must handle

- **401** — the session expired. Re-run `login` once and retry the request. Do not loop.
- **503** — the array cannot serve the management API right now. Back off and retry.
- **403 with `error.code == REMOTE_PERMISSION_REQUIRED`** — you crossed into a replication peer.
  You need that peer's credentials on the `X-Remote-Authorization` header (Basic). Do not retry
  blindly.

Errors arrive on `error` as `{code, message, severity, reasons}`. There is no
`application/problem+json` — do not expect RFC 9457 shapes. See
`errors/infinidat-problem-types.yml`.
