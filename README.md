# Infinidat

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Infinidat is an enterprise storage vendor — a Lenovo company since April 2026 — building
petabyte-scale primary storage (InfiniBox, InfiniBox SSA), backup and rapid-recovery appliances
(InfiniGuard), and the InfiniSafe cyber-resilience layer. Every InfiniBox array serves its own
management REST API, "InfiniAPI", at `https://<array>/api/rest`, alongside the InfiniShell CLI
and an HTML5 GUI.

- Website: https://www.infinidat.com/en
- Support / documentation: https://support.infinidat.com/hc/en-us
- GitHub: https://github.com/Infinidat
- InfiniSDK: https://infinisdk.readthedocs.io/en/latest/

## The one thing to know

The InfiniBox API is **appliance-local**. There is no public Infinidat API endpoint, no hosted
base URL, no sandbox and no self-service signup — the base URL is the customer's own array.

## What is in this repo

| Path | What it is |
|---|---|
| `postman/` | Infinidat's own **InfiniBox 7.3 API** Postman collection, saved verbatim from [Infinidat/api_7_3](https://github.com/Infinidat/api_7_3) — 47 requests |
| `openapi/` | An OpenAPI 3.1 **derived** from that collection: 40 operations across 38 paths |
| `overlays/` | Records exactly which parts of the OpenAPI are Infinidat's and which are ours |
| `conventions/` | Pagination, filtering, field selection, the result/metadata/error envelope, the `?approved=true` approval gate, the `x-infinidat-deprecated-api` header |
| `errors/` | Error envelope, status semantics, and the error codes observable in public first-party sources |
| `authentication/` | Session cookie + HTTP Basic profile |
| `data-model/` | 30 entities, 20 relationships; flags which are confirmed in the spec and which are only documented elsewhere |
| `asyncapi/` | The event surface — queryable events, SNMP/SYSLOG notification targets. **No AsyncAPI, no webhooks** |
| `packages/` | InfiniSDK, the Ansible collection, the CSI driver, and the registries that came back empty |
| `cli/` | InfiniShell |
| `changelog/` | Dated release history for the Ansible collection and InfiniSDK |
| `lifecycle/` | Versioning, the deprecation mechanism, and the dead `code.infinidat.com` developer portal |
| `conformance/` | CSI, Cinder, FIPS 140-2, SAML, LDAP — and the API standards Infinidat does *not* implement |
| `mcp/` | A **derived candidate** tool set. Infinidat publishes no MCP server |
| `skills/` | Four agent skills grounded in real operationIds |
| `well-known/` | Probe results — nothing published |
| `security/` | Domain security probe (TLS, HSTS, SPF, DMARC, DNSSEC, CAA) |
| `llms/` | Generated llms.txt |

## Known gaps

No OpenAPI from the vendor, no AsyncAPI, no webhooks, no GraphQL, no gRPC, no MCP server, no A2A
agent card, no `/.well-known/` surface, no `security.txt`, no public vulnerability-disclosure
policy, no trust center, no status page, no public API changelog, no sandbox. The API reference
and the full error/event code registries are behind the support-portal login.
