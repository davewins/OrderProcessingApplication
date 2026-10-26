---
name: pattern-rest-to-ems
description: BW6 integration pattern — accept a REST request, map to a canonical schema, publish to EMS/JMS, and return HTTP 202 Accepted for asynchronous downstream processing. Use when the user is designing or reviewing an API that fires-and-forgets work onto a queue/topic (order intake, event ingestion, async command bus). Trigger on phrases like "REST to EMS", "REST to JMS", "async intake", "publish and 202", "accept and enqueue".
---

# Pattern: REST → Canonical → EMS Publish → 202

## Intent
Accept an inbound REST call, transform the JSON payload to a **canonical** internal schema, publish it to an EMS (JMS) destination, and immediately return **HTTP 202 Accepted**. The caller does not wait for downstream processing.

## Typical flow
```
HTTP Client → REST Binding (POST) → JSON→Canonical Mapper → JMS Send (EMS) → 202 Accepted
                                                              │
                                                              └── (async) EMS subscriber processes the message
```

## Primary use case
Accept an API request and process it asynchronously. Good fit when:
- The downstream work is slow, batchy, or performed by a different team/service.
- You need to decouple the caller's SLA from downstream availability.
- Multiple consumers may fan out from the same event (topic).

## BW6 implementation

### Palette activities
| Step | Activity (palette) | Notes |
|---|---|---|
| Receive REST | **REST → REST Service Binding** (in Service descriptor) | Bound to a Swagger operation. HTTP method = POST. |
| Validate & map | **General Activities → Mapper** (input mapping on JMS Send) | Map JSON model → canonical XSD element. |
| Publish | **JMS → JMS Send Message** (Message Type = TextMessage or MapMessage) | Point at an EMS `JMSConnection` shared resource; set `DestinationType = Queue` or `Topic`. |
| Reply | **REST binding reply** (implicit) | Return `202` with a small ack body (e.g. `{ "status": "accepted", "correlationId": "..." }`). |

### Shared resources
- **HTTP Connector** for the REST service (with TLS — see `bw6-rules/SSLServerConnectorShouldHaveTLSprotocol.md`).
- **JMSConnection** shared resource pointing at the EMS server. **Never hardcode host/port/user** — bind them to Module Properties (`bw6-rules/JMSHardCoded.md`, `bw6-rules/BwSharedResourceUsingModuleProperty.md`).
- Canonical XSD stored under `Schemas/` in the module.

### Error handling
- Wrap `JMS Send Message` in a **scoped group with a Catch** so a JMS outage returns a controlled `503 Service Unavailable` instead of leaking a stack trace.
- Do **not** checkpoint here unless the pattern is upgraded to store-and-forward (see `bw6-rules/CheckpointProcessREST.md` for when a checkpoint is appropriate).
- Log the correlation id on both success and failure paths.

## Key validation question
> **Must EMS publication succeed before returning 202?**

Two variants — pick one and be explicit in design:

1. **Publish-then-202 (default, safest).** `JMS Send` runs synchronously in the request thread. On publish failure, return `503` (or `500`). Caller can retry safely if the API is idempotent.
2. **Fire-and-forget 202.** Return `202` first, publish on a background/subprocess. Higher throughput, but you **must** persist the payload (DB, file, checkpoint) before replying, or you will lose messages on a broker outage. Only choose this with an explicit durability story.

Ask the user which variant they want before wiring the process.

## Design checklist
- [ ] REST operation returns `202` (not `200`) and documents the async contract in the Swagger.
- [ ] JMS destination name, connection factory, and credentials come from **module properties**, not literals.
- [ ] Canonical schema is versioned and lives in the module's `Schemas/` folder.
- [ ] Catch handler on JMS Send converts broker failures to a defined HTTP fault.
- [ ] Correlation id is generated (or extracted from a header) and echoed in the response body.
- [ ] TLS enabled on the HTTP Connector; JMS connector uses `confidentiality` where required (`bw6-rules/JMSConnectorShouldHaveConfidentiality.md`).
- [ ] Process has a test suite covering happy path and JMS-down path (`bw6-rules/ProcessWithoutTest.md`).

## Authoring in BW6
Use the [[bw6design]] skill for the actual `bwdesign` CLI / MCP commands to create the process, add the REST service, the JMS Send activity, and the mapper. Apply the design conventions in [[bw6-rules]] (`RULES.md`, `METRICS.md`).
