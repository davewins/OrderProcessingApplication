---
name: pattern-source-to-ems-publish
description: BW6 integration pattern — read from a source system (DB polling, file watcher, scheduled fetch, etc.), transform to a canonical schema, and publish to an EMS topic/queue for downstream subscribers. Use when the user is designing a producer-side pub-sub or change-data-capture flow. Trigger on phrases like "publish to EMS", "publish to JMS", "source to queue", "CDC to topic", "outbound event stream", "pub-sub producer".
---

# Pattern: Source System → Canonical → EMS Publish

## Intent
Detect or fetch new data from a **source system** (database, file drop, external API, timer), normalize it to a **canonical event schema**, and publish it to **EMS (JMS)** so any number of subscribers can consume independently. This is the *producer* side of a pub-sub / event-stream integration.

## Typical flow
```
Source (poll / trigger / watcher) → Read/Fetch → Mapper (source → canonical) → JMS Send (EMS Topic or Queue)
                                                                                   │
                                                                                   └── (async) N subscribers consume
```

## Primary use case
Pub-sub — publish domain events so downstream systems can react without direct coupling. Common variants:
- **CDC-style:** poll a DB table (or a change table / outbox) on a timer, publish each new row as an event.
- **File watcher:** new file lands in a directory → parse rows → publish one event per record.
- **Scheduled fetch:** call an upstream API on a cron, publish diffs.
- **Trigger-driven:** upstream calls a BW6 REST/SOAP endpoint that internally fans out to EMS.

## BW6 implementation

### Palette activities
| Step | Activity (palette) | Notes |
|---|---|---|
| Trigger | **Timer** (scheduled), or **JDBC Query** in a poller subprocess, or **File Poller**, or **REST/SOAP receive** | Only **one** starter per process (`bw6-rules/AtLeastOneStarter.md`). |
| Fetch | **JDBC Query** / **Read File** / **Send HTTP Request** / **Parse XML/CSV** as appropriate | For CDC, track a high-water mark (last processed id/timestamp) in a persistent store — not in memory. |
| Iterate | **General Activities → Group** with `Group Type = Iterate` | One iteration per record if you got a batch. |
| Map | **Mapper** on the JMS Send input | Source model → canonical XSD. |
| Publish | **JMS → JMS Send Message** | `DestinationType = Topic` for classic pub-sub; `Queue` for competing-consumer producer. |
| Bookkeeping | JDBC Update to advance the high-water mark | Do this **only** after successful publish. |

### Shared resources
- **JMSConnection** shared resource for EMS — host, factory, credentials from module properties (`bw6-rules/JMSHardCoded.md`).
- **JDBC Connection** (if source is a DB) with driver/URL/creds from module properties.
- Canonical XSD in `Schemas/`, versioned.
- File resource (if source is a file drop).

### Publishing rules
- Publish messages as **persistent** if downstream reliability matters (default: persistent). Non-persistent only for high-volume, loss-tolerant streams — call it out explicitly and see `bw6-rules/JMSRequestReplyNonPersistent.md`.
- Set a **JMSType** / message property that identifies the event schema and its version (subscribers rely on this to route/deserialize).
- Set a **correlation id** on each message so downstream logs can be joined back to a source record.

### Ordering and duplicates
- Topics do not guarantee ordering across subscribers unless you use a partitioned/unified subject. If subscribers depend on order, publish to a **single queue** and use a message group id.
- At-least-once is the default: duplicates are possible on retry. Subscribers must be idempotent — see `pattern-ems-subscribe-to-target` for the consumer side.

### Error handling
- Wrap `JMS Send` in a scoped group with a **Catch**. On failure, do **not** advance the high-water mark — the next poll will retry.
- Log the source key (row id / filename / offset) on both success and failure.
- Consider `Checkpoint` if the process is long-lived and you need crash-recovery mid-batch (`bw6-rules/CheckpointProcessJDBC.md`).

## Key validation question
Not filled in the source spreadsheet. Ask the user these before wiring:

- **What guarantees the source is not re-read?** (High-water mark table? File move-to-`processed/`? Outbox flag update?)
- **Topic or queue?** Multiple independent subscribers → topic. One competing pool → queue.
- **Ordering required?** If yes, single queue + message group; not a topic.
- **Persistence?** Persistent messages unless there's a stated reason not to.
- **Batch size / poll interval** — must be tuned to source volume and downstream capacity.

## Design checklist
- [ ] Exactly one starter activity on the process.
- [ ] Source position (high-water mark, offset, processed-file marker) is stored durably, not in memory.
- [ ] Position is advanced **after** publish succeeds — never before.
- [ ] Canonical XSD is versioned; version identifier is on the JMS message (header / property).
- [ ] JMS connection details from module properties, not literals.
- [ ] Publish is persistent unless explicitly waived.
- [ ] Correlation id set on every message.
- [ ] Retry semantics: at-least-once is documented; subscribers know they must dedupe.
- [ ] Tests cover: nothing-new, one new record, batch of N, JMS-down (nothing advances), source-down.

## Authoring in BW6
Use [[bw6design]] to create the starter (Timer / JDBC poller / File poller / REST), the JMS Send activity, and the mapper. Follow [[bw6-rules]] — `JMSHardCoded.md`, `JMSAcknowledgementMode.md`, `AtLeastOneStarter.md`, `BwSharedResourceUsingModuleProperty.md`.

Pairs with [[pattern-ems-subscribe-to-target]] on the consumer side.
