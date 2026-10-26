---
name: pattern-ems-subscribe-to-target
description: BW6 integration pattern — subscribe to an EMS topic/queue, transform the canonical message, and write it into a target system (database, REST API, SOAP service, file, ERP). Use when the user is designing a consumer-side pub-sub flow, an event-to-system integration, or a message-driven writer. Trigger on phrases like "EMS subscriber", "JMS receiver", "consume from queue", "queue to database", "pub-sub consumer", "message-driven writer".
---

# Pattern: EMS Subscribe → Target System

## Intent
Consume messages from an **EMS (JMS)** destination, transform to the **target system's** contract, and apply them (insert row, call REST, invoke SOAP, write file, etc.). This is the *consumer* side of a pub-sub / async-command flow.

## Typical flow
```
JMS Receive (Queue/Topic subscriber) → Mapper (canonical → target) → Target activity (JDBC / HTTP / SOAP / File) → [Ack / Confirm]
                          │
                          └── failure → retry / dead-letter
```

## Primary use case
Pub-sub — consume domain events (or async commands) and land them in a target of record. Common variants:
- **Queue → DB writer:** each message becomes an INSERT/UPDATE.
- **Queue → REST invoke:** relay the event to a downstream API.
- **Topic subscriber:** react to broadcast events (audit, projection, notification).
- **Command bus:** the front side of a REST→EMS pattern; this side does the actual work.

## BW6 implementation

### Palette activities
| Step | Activity (palette) | Notes |
|---|---|---|
| Subscribe | **JMS → JMS Receive Message** (as a process **starter**) | For topics needing survive-restart, use a **durable subscriber** — set `SubscriptionName` and use a client id. |
| Ack strategy | On the JMS Receive activity: `Acknowledgement Mode` | `AUTO` is easiest but risks message loss on downstream failure. Use `CLIENT` (with a **JMS Confirm** activity) or `TRANSACTIONAL` when a target write must succeed before ack. See `bw6-rules/JMSAcknowledgementMode.md`, `bw6-rules/JMSReceiverPlusConfirm.md`. |
| Map | **Mapper** on the target activity input | Canonical XSD → target model. |
| Write | **JDBC Update**, **Invoke REST API**, **Invoke SOAP**, **Write File**, or vendor-specific activity | Wrap in a transaction group if the target is a DB and you need all-or-nothing. |
| Confirm | **JMS → JMS Confirm** (when using `CLIENT` ack) | Only after the target write succeeds. |
| Failure | **Catch** on write → decide retry / dead-letter / rollback | See error handling below. |

### Shared resources
- **JMSConnection** shared resource for EMS — from module properties (`bw6-rules/JMSHardCoded.md`).
- **JDBC Connection** / **HTTP Client** / etc. for the target — also property-bound.

### Ack + delivery guarantees
- **AUTO ack:** message is ack'd on receive. If the target write fails, the message is gone. Only use for tolerable/derivable events.
- **CLIENT ack + Confirm after write:** message stays on the broker until Confirm. On failure, redeliver.
- **TRANSACTIONAL:** JMS receive + DB write in one XA transaction. Strongest guarantee, but requires broker + DB XA capability, and adds latency. Use only when duplicates are unacceptable AND the target supports XA.
- Default: **CLIENT ack, Confirm after successful write**.

### Idempotency (essential)
- The publisher side is at-least-once. **This consumer WILL see duplicates.**
- Dedupe using either:
  1. A **unique key** on the target (`message id` column with a unique index; duplicate insert → catch & drop).
  2. An **idempotency table** keyed by `JMSMessageID` or the domain event id.
- Do not rely on "it usually doesn't duplicate" — it will, at the worst possible moment.

### Retry and dead-letter
- Set a max redelivery count on the destination (broker side).
- On the process side: on target-write failure, let the exception propagate so the message is redelivered — do **not** silently ack.
- Configure a **dead-letter queue** for messages that fail beyond the retry limit. Include the last exception in a message property when routing to DLQ, so ops can triage.
- Do not loop-retry inside a single message handler for hours — you'll stall the consumer.

### Concurrency
- The JMS Receive activity's session count / prefetch controls parallelism. Tune to target write capacity — flooding the DB with 50 parallel writers is a common mistake.
- Ordering: if the publisher used a **message group id**, ensure the receiver preserves per-group ordering (single consumer per group).

## Key validation question
Not filled in the source spreadsheet. Ask the user these before wiring:

- **Delivery guarantee required?** At-least-once (default) / exactly-once (XA) / at-most-once (rare).
- **Dedupe strategy?** Which field is the idempotency key, and where is it stored?
- **On terminal failure — DLQ or fail-loud?** How is the DLQ monitored?
- **Ordering constraints?** If yes, message groups and single-consumer-per-group are required.
- **Durable subscriber?** (For topics — do you need to catch up on messages published while the subscriber was down?)

## Design checklist
- [ ] JMS connection details from module properties, not literals.
- [ ] Acknowledgement mode chosen deliberately (documented) — not left at default.
- [ ] If `CLIENT` ack, a **JMS Confirm** is present after the successful write (`bw6-rules/JMSReceiverPlusConfirm.md`).
- [ ] Idempotency mechanism is in place — duplicates cannot corrupt the target.
- [ ] Target write is in a transaction group (if DB) so partial writes roll back.
- [ ] Broker redelivery limit + DLQ are configured.
- [ ] Concurrency (session count) tuned to target capacity.
- [ ] Correlation id from the message is logged on both success and failure paths.
- [ ] Tests cover: happy path, duplicate message (idempotent), target down (redelivery), poison message (DLQ).

## Authoring in BW6
Use [[bw6design]] to add the `JMS Receive Message` starter, the target-write activity, and (if using CLIENT ack) the `JMS Confirm`. Follow [[bw6-rules]] — `JMSAcknowledgementMode.md`, `JMSReceiverPlusConfirm.md`, `JMSHardCoded.md`, `AtLeastOneStarter.md`, `ExceptionHandlingCheck.md`.

Pairs with [[pattern-source-to-ems-publish]] on the producer side.
