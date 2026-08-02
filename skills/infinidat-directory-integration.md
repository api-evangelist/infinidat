---
name: Configure and verify an InfiniBox LDAP / Active Directory repository
description: Register, test, reorder and update the LDAP or Active Directory user repositories an InfiniBox array authenticates against, using the array's own connectivity, domain-resolution and group-membership test operations before committing a change.
api: openapi/infinidat-infinibox-openapi.yml
operations:
  - login
  - ldapGetRepositories
  - ldapGetFirstRepository
  - ldapTestRepositoryComminucation
  - ldapCreateAUserRepository
  - ldapTestLDAPByID
  - ldapTestGroupByLDAPID
  - ldapResolveDomain
  - ldapUpdateTheRepositoryAttributes
  - ldapReorder
  - ldapReload
  - logout
generated: '2026-08-01'
method: generated
source: openapi/infinidat-infinibox-openapi.yml
---

# Configure and verify an InfiniBox LDAP / Active Directory repository

**This skill changes who can log in to a storage array. A bad change here locks every
administrator out of the box.** Treat every write step as requiring human confirmation, and never
run the reorder or update steps unattended.

The array gives you test operations for exactly this reason — it will let you validate a
repository *before* and *after* you commit it. Use them.

## Steps

1. **Authenticate.** `login` — `POST /users/login`.
2. **Read what is there now.** `ldapGetRepositories` — `GET /config/ldap/`. Record the current
   list and its order. `ldapGetFirstRepository` — `GET /config/ldap/{ldap_id}` reads one.
   **Save this state.** It is your rollback.
3. **Dry-run the connection before creating anything.** `ldapTestRepositoryComminucation` —
   `POST /config/ldap/test`. This tests a repository definition without persisting it. If this
   fails, stop — do not create the repository and hope.
4. **Resolve the domain.** `ldapResolveDomain` — `POST /config/ldap/resolve_domain`. Confirms the
   array can discover the domain controllers for the domain you gave it.
5. **Create it.** `ldapCreateAUserRepository` — `POST /config/ldap`. Capture the returned id.
6. **Test the persisted repository.** `ldapTestLDAPByID` — `POST /config/ldap/{ldap_id}/test`.
7. **Test group resolution.** `ldapTestGroupByLDAPID` — `POST /config/ldap/{ldap_id}/test_group`.
   A repository that connects but cannot resolve the admin group grants nobody anything.
8. **Only then adjust precedence.** `ldapReorder` — `POST /config/ldap/set_order`. Order decides
   which repository resolves a username first. Changing it can silently re-bind existing
   administrators to different accounts.
9. **Update attributes if needed.** `ldapUpdateTheRepositoryAttributes` —
   `PUT /config/ldap/{ldap_id}`.
10. **Reload.** `ldapReload` — `POST /config/ldap/reload` to make the array pick the change up.
11. **Verify you can still get in.** Open a second, independent session with `login` before you
    close the one you have. Do not `logout` from your working session until that succeeds.
12. **Log out.** `logout` — `POST /users/logout`.

## The approval gate

InfiniBox gates high-consequence operations behind an explicit approval flag. If a call returns
an `error.code` of `APPROVAL_REQUIRED` or `APPROVAL_REQUIRED_VOLUME_HAS_CHILDREN`:

1. Read `error.reasons[0]` — that is the array's own plain-language explanation of what you are
   about to do.
2. **Show it to a human and get a decision.** Do not auto-approve.
3. Re-issue the *identical* request with `?approved=true`.

This is the closest thing InfiniBox has to a human-in-the-loop primitive, and it exists precisely
for the class of operation this skill performs. Suppressing it defeats the control.

## Other errors

- **401** — session expired; re-login once and retry.
- **403 with `error.code == REMOTE_PERMISSION_REQUIRED`** — the operation reached a replication
  peer; supply that system's credentials on `X-Remote-Authorization` (Basic).
- **503** — the array is not serving the management API; back off and retry.

## Automation alternative

If you are configuring directories as part of a repeatable build rather than a one-off change,
the `infinidat.infinibox` Ansible collection's `infini_users_repository`, `infini_user` and
`infini_sso` modules wrap these same operations with declarative `state: present|absent`
semantics — which is where idempotency lives in the Infinidat ecosystem, since the REST API
itself has no idempotency-key contract.
