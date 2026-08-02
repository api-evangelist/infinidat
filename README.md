# Infinidat

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
