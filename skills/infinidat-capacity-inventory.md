---
name: Inventory InfiniBox pools, filesystems and their metadata
description: Build a capacity and tenancy inventory of an InfiniBox array by listing storage pools and filesystems, resolving a pool by name or id, and reading the key/value metadata tagged onto each storage entity.
api: openapi/infinidat-infinibox-openapi.yml
operations:
  - login
  - poolGetPools
  - fsGetAllFilesystems
  - fsGetFilesystemByID
  - metatdata
  - metatdataGetByObject
  - systemGetTotalVirtualCapacity
  - logout
generated: '2026-08-01'
method: generated
source: openapi/infinidat-infinibox-openapi.yml
---

# Inventory InfiniBox pools, filesystems and their metadata

Read-only. Use this to answer "who is using what capacity on this array".

## Steps

1. **Authenticate.** `login` — `POST /users/login`.
2. **Establish the ceiling.** `systemGetTotalVirtualCapacity` —
   `GET /system/capacity/total_virtual_capacity`. Every pool number below is a share of this.
3. **List pools.** `poolGetPools` — `GET /pools`. Page with `page` / `page_size` until
   `metadata.page == metadata.pages_total`, and read `metadata.number_of_objects` for the total.
4. **Resolve a specific pool.** The same operation is how you look one up — the array supports
   both `GET /pools?id=358` and `GET /pools?name=eq:my-pool`. Prefer the filter form
   (`name=eq:...`) over a client-side scan.
5. **List filesystems.** `fsGetAllFilesystems` — `GET /filesystems`, again paged.
6. **Read one filesystem.** `fsGetFilesystemByID` — `GET /filesystems/{id}`.
7. **Read the metadata tags.** `metatdata` — `GET /metadata/` for everything, or
   `metatdataGetByObject` — `GET /metadata/{object_id}` for one entity. InfiniBox lets
   operators attach arbitrary key/value metadata to any storage entity, and in practice that is
   where tenancy, cost-centre, application-owner and ticket references live. An inventory that
   ignores metadata will not tell you who owns the capacity.
8. **Log out.** `logout` — `POST /users/logout`.

## Field selection is what makes this fast

An unqualified `GET /filesystems` on a large array returns a lot. Ask for what you need:

```
GET /filesystems?fields=id,name,size,used_size,pool_id&page_size=1000&sort=name
```

`fields` is comma-separated; `page_size` maxes at 1000; `sort` takes a field name and accepts a
leading `-` to reverse.

## Cautions

- **Do not treat this as a billing source of record.** These are live array numbers with no
  point-in-time guarantee, and there is no ETag or version on a read.
- **Volumes are not covered by the captured spec.** The 7.3 collection this API description was
  derived from exercises pools and filesystems but not the block-volume path — a request in the
  collection named "Volumes" actually points at `/metadata/`. If you need block volumes, use
  InfiniSDK's `system.volumes` or the `infini_vol` Ansible module rather than guessing a path.
  See `data-model/infinidat-data-model.yml` for what is and is not confirmed in the spec.
- **No idempotency key exists** on this API. That does not matter for these reads, but it does
  the moment you follow this skill with a write.
