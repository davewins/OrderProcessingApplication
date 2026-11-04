# OrderProcessing

> Generated from the BusinessWorks project by the DevHub import flow. Regenerate by re-running
> the import, or edit these pages by hand.

TIBCO BusinessWorks Container Edition application OrderProcessing 1.0.0 with 11 process(es) providing the Orders API.

| | |
|---|---|
| Application | OrderProcessing |
| Application version | 1.0.0 |
| BusinessWorks version | 2.10.0 HF2 V49 2025-03-21 |
| Edition | Container Edition (BWCE) |
| Processes | 11 |
| Activities | 23 |
| REST operations | 4 |
| Messaging destinations | 3 |
| API descriptors | 1 |
| Shared resources | 2 |
| Schemas | 7 |
| Test files | 1 |
| Findings | 4 (3 warnings) |
| Owner | group:default/tibco-imported |
| System | order-management-system |
| Lifecycle | production |

## What it does

- Exposes **4 REST operations** (see [API](api.md)).
- Consumes **StockCheck**: StockCheck.
- Sends to **StockCheck**: InventoryCheck.
- Consumes **LogMessage**: AuditLog.
- Sends to **LogMessage**: Logger.
- Sends to **Orders**: CreateOrderProcess.
- Is made up of 11 processes; the [process page](processes.md) shows what triggers each one and what it calls.


## Pages

- [Processes](processes.md): triggers, call tree and per-process activity summary
- [Messaging](messaging.md): queues and topics with their producers and consumers
- [API](api.md): endpoints and the process that implements each
- [Resources and configuration](resources.md): shared resources, external dependencies and variables
- [Schemas](schemas.md): the data contracts
- [Findings](quality.md): configuration and process issues worth a second look

## Palettes used

- `bw.generalactivities`
- `bw.jms`
- `bw.httpconnector`
- `bw.jmsconnection`
- `bw.jndiconfiguration`
- `bw.rest`

