---
name: jmsRetailOrderQueue
description: Build the "RetailOrderQueue" BW6 application — a JMS Receive Message starter (Queue) that logs incoming orders, appends them to a file, and replies over JMS. Use when the user asks to create/scaffold the retail JMS order queue example, needs a BW6 JMS palette sample with JNDI + JMS Connection shared resources, or references any of: "retail order queue", "RetailOrderQueue", "JMS Receive Message", "Reply to JMS Message", "TibjmsInitialContextFactory", "retail.orders.queue", "EMSConfig", "RetailJMSConnection". Drives the build via `bwdesign` / `mcp__bw__*` tools and applies `bw6-rules` checks.
---

# RetailOrderQueue — JMS Palette Sample (BW6)

Introduces **JNDI**, **JMS Connection**, **JMS Receive Message** starter, and **Reply to JMS Message** — the queue request/reply pattern against TIBCO EMS.

Category: **JMS** • Main tech: `JNDI, JMS Connection, JMS Receive Message, Reply to JMS Message`.

## How to run this skill

1. Confirm the target BW workspace with the user. Prefer `mcp__bw__*` if Business Studio is open, else `bwdesign` per the `bwdesign` skill.
2. Execute the spec below step by step. Announce each major step.
3. Cross-check against `bw6-rules`. Rules to watch here:
   - `JMSAcknowledgementMode` — verify the acknowledgement mode. If CLIENT_ACK is selected, also add a `Confirm` activity on every OK flow (`JMSReceiverPlusConfirm`).
   - `JMSHardCoded` — the sample already binds `Destination` to a module property; do the same for `Timeout`, `Reply Destination`, `Message Selector`, `Polling Interval` if you set them.
   - `JMSConnectorShouldHaveConfidentiality` — enable SSL confidentiality on the JMS Connector when targeting a secure EMS.
   - `LastActivityAndEndActivity` — the `Reply to JMS Message` correctly terminates the flow.
4. Validate and report status.

## Project Specification

### Project Hierarchy

| Component Type | Name |
| :---- | :---- |
| **Application Module** | `RetailOrderQueue` |
| **Application Project** | `RetailOrderQueue.application` |

### Module Properties

| Property Name | Data Type | Value |
| :---- | :---- | :---- |
| QUEUE_NAME | String | `retail.orders.queue` |
| OUTPUT_FILE | String | `C:\tmp\RetailOrders\orders.log` |

### Shared Resources

**JNDI — `EMSConfig`** (FQN `retail.jms.EMSConfig`)
- `Provider` = `TIBCO EMS`
- `Initial Context Factory` = `com.tibco.tibjms.naming.TibjmsInitialContextFactory`
- `Provider URL` = `tibjmsnaming://localhost:7222`

**JMS Connection — `RetailJMSConnection`** (FQN `RetailOrderQueue.RetailJMSConnection`)
- `Messaging Style` = `Queue`
- `JNDI Configuration` = `retail.jms.EMSConfig`

### Process `ReceiveRetailOrder.bwp`

Activities: `JMS Receive Message` → `Log` → `Write File` → `Reply to JMS Message`. Link in sequence.

- **JMS Receive Message (Starter)**
  - `JMS Connection` = `RetailOrderQueue.RetailJMSConnection`
  - `Messaging Style` = `Queue`
  - `Destination` = module property `QUEUE_NAME`
- **Log** — `message` = `concat("Retail order received: ", $JMSReceiveMessage/Body)`
- **Write File**
  - `FileName` = module property `OUTPUT_FILE`
  - `Append` = `true`
  - `Create Non Existing Directories` = `true`
- **Reply to JMS Message** — `Body` = `"Retail order processed successfully"`
