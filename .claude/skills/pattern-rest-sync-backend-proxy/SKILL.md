---
name: pattern-rest-sync-backend-proxy
description: BW6 integration pattern — expose a simplified REST API that synchronously proxies to an existing backend (REST or SOAP), transforming the request/response between the public contract and the backend. Use when the user is designing an API facade, BFF (backend-for-frontend), or a strangler layer over a legacy service. Trigger on phrases like "API facade", "backend proxy", "REST wrapper", "strangler", "expose backend as REST".
---

# Pattern: REST Synchronous Backend Proxy

## Intent
Present a **clean, purpose-built REST API** to clients while delegating the real work to an existing backend. The BW6 process transforms both directions — request into what the backend expects, backend response into what the client contract promises.

## Typical flow
```
Client → REST Binding (in) → Transform (req) → HTTP/SOAP Invoke (backend) → Transform (resp) → REST Reply (out)
                                                        │
                                                        └── fault path → Fault Mapper → REST error response
```

## Primary use case
Expose a simplified API over an existing backend. Common drivers:
- Backend is verbose, chatty, or uses a legacy protocol (SOAP, custom XML).
- You want to shrink the surface area, hide internal fields, or apply auth/quotas at the edge.
- Multiple backends need to be composed into one call (fan-out/fan-in — often extends this pattern).

## BW6 implementation

### Palette activities
| Step | Activity (palette) | Notes |
|---|---|---|
| Receive REST | **REST Service Binding** on the process | Bound to a Swagger operation. |
| Map to backend | **Mapper** (input mapping on the invoke activity) | Public JSON → backend model. |
| Call backend (REST) | **HTTP → Send HTTP Request** or **REST → Invoke REST API** | Prefer *Invoke REST API* when a Swagger for the backend is available — you get typed input/output. |
| Call backend (SOAP) | **SOAP → Invoke SOAP** | Requires a WSDL shared resource. |
| Map response | **Mapper** on the reply | Backend response → public JSON. |
| Reply | REST binding reply | 200 on success; mapped status codes on error. |

### Shared resources
- **HTTP Client** shared resource for the backend base URL (endpoint bound to a module property — `bw6-rules/EndpointURIFromHTTPBindingSetUsingProperty.md`).
- **HTTP Connector** for the exposed service (TLS on — `bw6-rules/SSLServerConnectorShouldHaveTLSprotocol.md`).
- WSDL resource (SOAP variant) with the backend's WSDL imported into the module.

### Error handling
- Add a **Catch-All** on the invoke to intercept backend failures.
- Decide **pass-through vs normalize** (see validation question below) and implement one consistently.
- Do **not** checkpoint (`bw6-rules/CheckpointProcessREST.md`) — this is a synchronous, stateless call.
- Timeouts on the HTTP Client must be shorter than the client's expected SLA. Document them.

## Key validation question
> **Should backend faults be passed through or normalized?**

Two variants — decide upfront and don't mix:

1. **Pass-through.** Backend `4xx`/`5xx` and error body flow to the client largely unchanged (perhaps re-serialized). Simpler, but leaks backend semantics and couples clients to backend errors.
2. **Normalize.** Map backend faults to a small, documented set of API-level error codes (`{ "error": "UPSTREAM_UNAVAILABLE", "code": "E5001" }`). More work, but the API contract stays stable when the backend changes. **Recommended** for public-facing APIs.

Ask the user which they want; wire the Catch handler and Fault Mapper accordingly.

## Design checklist
- [ ] Public Swagger describes the API in *its own* terms — no backend field names leak.
- [ ] Backend endpoint URL comes from a **module property**, not a literal.
- [ ] HTTP Client timeout is set and shorter than upstream client's expected latency budget.
- [ ] Error strategy (pass-through vs normalize) is documented and implemented in exactly one place.
- [ ] Auth headers (Bearer, API-Key) added via the Mapper or a header set, not hardcoded.
- [ ] TLS enforced both inbound and outbound (`bw6-rules/HttpConnectorShouldHaveConfidentiality.md`, `bw6-rules/HttpClientSSLShouldHaveConfidentiality.md`).
- [ ] Process has tests for happy path, backend timeout, backend 4xx, backend 5xx.

## Authoring in BW6
Use [[bw6design]] to create the REST service binding, add the *Invoke REST API* / *Invoke SOAP* activity, and wire the mappers. Apply conventions from [[bw6-rules]] — pay special attention to `HttpClientMustBeUsedinHTTPBinding.md` and `EndpointURIFromHTTPBindingSetUsingProperty.md`.
