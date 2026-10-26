---
name: pattern-rest-to-soap-mediation
description: BW6 integration pattern — accept a REST/JSON request, invoke a legacy SOAP service, and return a REST response, so new REST consumers can use existing SOAP backends without needing WSDL/XSD tooling. Use when the user is designing a REST facade over a legacy SOAP service. Trigger on phrases like "REST to SOAP", "REST facade over SOAP", "expose SOAP as REST", "modernize SOAP consumers".
---

# Pattern: REST → SOAP Mediation

## Intent
Give **REST/JSON clients** access to an existing **SOAP** backend. BW6 accepts a REST call, builds the SOAP envelope, invokes the SOAP service, and translates the response (and any faults) back to REST.

## Typical flow
```
REST Client → REST Binding (in) → Mapper (JSON → XML) → SOAP Invoke → Mapper (XML → JSON) → REST Reply (2xx)
                                                            │
                                                            └── SOAP Fault → Fault Mapper → REST error (4xx/5xx)
```

## Primary use case
Expose legacy SOAP services as REST APIs. Typical drivers:
- New mobile / SPA / partner clients that don't want to speak SOAP.
- API gateway standardization: everything on the gateway is REST regardless of what's behind it.
- Preparing to eventually retire the SOAP service — introduce the REST contract first, migrate consumers, then swap the backend.

## BW6 implementation

### Palette activities
| Step | Activity (palette) | Notes |
|---|---|---|
| Receive REST | **REST Service Binding** | Bound to a Swagger operation you author. |
| Map to SOAP | **Mapper** on the SOAP invoke input | JSON → XSD elements defined by the backend WSDL. |
| Call SOAP | **SOAP → Invoke SOAP** | Requires the backend **WSDL** as a shared resource. |
| Map response | **Mapper** on the REST reply | XML → JSON. Drop backend-only fields; rename to REST-friendly camelCase. |
| Fault path | **Catch** on Invoke SOAP, **Reply with error** | Translate SOAP faults into HTTP status + JSON error body. |

### Shared resources
- **WSDL** for the backend SOAP service, imported into the module.
- **HTTP Client** shared resource for the SOAP endpoint (URL bound to a module property).
- **HTTP Connector** for the exposed REST service (TLS on).

### Contract design
- Do **not** make the REST payload a mechanical mirror of the SOAP XSD. That defeats the purpose. Design a REST resource model in *client terms* first, then map.
- JSON field names: `camelCase`. Enums: uppercase strings, not the numeric codes SOAP often uses.
- Drop SOAP wrapper elements from the REST response (`ResponseEnvelope > ResponseBody > Payload > ...` should collapse to just the payload).

## Key validation question
> **Should the REST contract hide the SOAP data model completely?**

Two variants — decide upfront:

1. **Full abstraction (recommended for external / public APIs).** The REST contract is a clean, purpose-built resource model. Field names, structure, error codes, and enums are chosen for REST/HTTP conventions. Backend can be replaced later without breaking clients.
2. **Thin translation (fine for internal / short-lived APIs).** JSON is a near-1:1 rendering of the SOAP XSD (elements → objects, attributes → fields). Fast to build, but the REST clients are effectively coupled to the SOAP schema and will break the day the SOAP service changes.

Ask the user which one applies before designing the Swagger.

### SOAP fault → HTTP status mapping
Common mapping table — adjust to the specific WSDL faults:

| SOAP fault | REST response |
|---|---|
| Validation fault | `400 Bad Request` |
| Auth fault | `401 Unauthorized` or `403 Forbidden` |
| Not-found fault | `404 Not Found` |
| Business rule / conflict fault | `409 Conflict` |
| Throttle | `429 Too Many Requests` |
| Server fault / timeout | `502 Bad Gateway` (upstream) or `504 Gateway Timeout` |

Body shape (recommended): `{ "error": "<CODE>", "message": "<human msg>", "correlationId": "..." }`.

## Design checklist
- [ ] Backend WSDL endpoint URL is a module property.
- [ ] REST Swagger is designed in resource/client terms, not as a mirror of the WSDL.
- [ ] JSON field naming, casing, and enums follow REST conventions.
- [ ] SOAP fault → HTTP status mapping table is documented and every declared fault is handled.
- [ ] SOAP `faultstring` is not leaked verbatim to REST clients when it contains internal detail.
- [ ] Auth: REST `Authorization: Bearer` header → WS-Security / SOAP header injection in the mapper.
- [ ] TLS on both sides (`bw6-rules/SSLServerConnectorShouldHaveTLSprotocol.md`, `bw6-rules/HttpClientSSLShouldHaveConfidentiality.md`).
- [ ] `Content-Type: application/json` on all REST responses, including errors.
- [ ] Tests cover: happy path, each SOAP fault mapped, backend timeout, malformed SOAP response.

## Authoring in BW6
Use [[bw6design]] — REST binding + Swagger operation on the inbound side, `Invoke SOAP` activity + WSDL shared resource on the outbound side. Follow [[bw6-rules]] — see `SwaggerValidation.md`, `BindingShouldHavePolicyAssociated.md`, `EndpointURIFromHTTPBindingSetUsingProperty.md`.
