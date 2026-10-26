---
name: pattern-soap-to-rest-mediation
description: BW6 integration pattern — accept a SOAP request, mediate to a modern REST backend, and return a SOAP response, so legacy SOAP consumers keep working while the backend is modernized. Use when the user is designing a modernization / strangler wrapper that must preserve an existing SOAP contract. Trigger on phrases like "SOAP to REST", "SOAP facade", "legacy SOAP wrapper", "keep SOAP contract".
---

# Pattern: SOAP → REST Mediation

## Intent
Preserve an existing **SOAP contract** for consumers while the real work is done by a **REST** backend. BW6 sits in the middle, translating SOAP↔REST and, critically, mapping REST error semantics back into SOAP faults.

## Typical flow
```
SOAP Client → SOAP Service Binding → Mapper (XML → JSON) → HTTP/REST Invoke → Mapper (JSON → XML) → SOAP Reply
                                                                    │
                                                                    └── HTTP error → Fault Mapper → SOAP Fault
```

## Primary use case
Modernize a backend without changing SOAP consumers. Typical when:
- The backend team has replaced a legacy service with a REST equivalent, but internal or external SOAP clients cannot be changed on that team's schedule.
- You want to phase migration: consumers switch off SOAP over months/years while the backend already lives on REST.

## BW6 implementation

### Palette activities
| Step | Activity (palette) | Notes |
|---|---|---|
| Receive SOAP | **Service** with a **SOAP Binding** (SOAP over HTTP) | Bound to an existing WSDL — do **not** regenerate it, or you'll break clients. |
| Map to REST | **Mapper** on the REST invoke input | XSD → JSON model. |
| Call backend | **REST → Invoke REST API** (preferred if Swagger is available) or **HTTP → Send HTTP Request** | Endpoint bound to a module property (`bw6-rules/EndpointURIFromHTTPBindingSetUsingProperty.md`). |
| Map response | **Mapper** on the SOAP reply | JSON → XSD, matching the original WSDL response element exactly. |
| Fault path | **Catch** on the invoke, **Reply with Fault** | Translate HTTP status/body into the WSDL-declared fault(s). |

### Shared resources
- Existing **WSDL** imported into the module. Preserve `targetNamespace`, operation names, message parts.
- **HTTP Connector** for the exposed SOAP service (TLS on).
- **HTTP Client** shared resource for the REST backend.

### Contract preservation
- Do **not** change the WSDL. If you must add a field, add it as an optional element at the end of the response type; regression-test existing clients.
- Response mapping must produce elements in the exact schema order — some SOAP stacks are order-sensitive.
- Namespaces on the response elements must match the WSDL — check `bw6-rules/DefaultTargetNamespace.md` and `bw6-rules/XMLResourceSameTargetNamespace.md`.

## Key validation question
> **How should REST errors map to SOAP faults?**

REST uses HTTP status + JSON body; SOAP uses `<soap:Fault>` with WSDL-declared fault types. You must decide the mapping table upfront:

| REST outcome | SOAP fault |
|---|---|
| `2xx` | Normal SOAP response |
| `400` validation error | `<soap:Fault>` with `faultcode = Client` and a WSDL-declared *ValidationFault* if one exists |
| `401` / `403` | `Client` fault, `AuthenticationFault` / `AuthorizationFault` |
| `404` (business not-found) | Either a WSDL-declared *NotFoundFault* or an empty-success response — pick one and be consistent |
| `409` conflict | `Client` fault, WSDL-declared *ConflictFault* |
| `429` | `Server` fault, `ThrottledFault` (or `Client` if you prefer clients to back off) |
| `5xx` / timeout | `Server` fault, `UpstreamFault` — never leak the backend body verbatim |

Ask the user to review this table before wiring the Catch handler. **Faults that aren't declared in the WSDL will not deserialize correctly on typed SOAP clients.**

## Design checklist
- [ ] The WSDL exposed to consumers is byte-identical to the pre-modernization WSDL.
- [ ] Backend REST endpoint URL is a module property.
- [ ] Response mapper produces elements in schema order with correct namespaces.
- [ ] REST-error → SOAP-fault table is documented and every branch is implemented in the Catch handler.
- [ ] Auth token forwarding: extract SOAP header (e.g. WS-Security / API-Key header) and inject into REST request as Bearer/API-Key.
- [ ] TLS on both the inbound SOAP connector and the outbound HTTP client (`bw6-rules/SSLServerConnectorShouldHaveTLSprotocol.md`, `bw6-rules/HttpClientSSLShouldHaveConfidentiality.md`).
- [ ] Tests cover: happy path, each declared fault type, backend timeout, malformed backend JSON.

## Authoring in BW6
Use [[bw6design]] — `system:createWSDL` / `system:setServicePortType` to expose the SOAP service, `system:createBinding` for the SOAP binding, and the REST-invoke activity for the backend call. Follow [[bw6-rules]] for `BindingShouldHavePolicyAssociated.md` and namespace/XSD rules.
