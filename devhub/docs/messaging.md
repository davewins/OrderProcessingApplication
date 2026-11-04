# Messaging

3 queues / topics used by the processes. Producers include request/reply senders.

| Destination | Kind | Configuration variable | Producers | Consumers |
|---|---|---|---|---|
| `StockCheck` | queue | - | InventoryCheck | StockCheck |
| `LogMessage` | queue | - | Logger | AuditLog |
| `Orders` | queue | - | CreateOrderProcess | - |


Broker: `tcp://192.168.1.30:9011`

## Messaging activities

| Process | Activity | Type | Destination | Connection |
|---|---|---|---|---|
| StockCheck | JMSReceiveMessage | `bw.jms.receive` | StockCheck | jmsConnection |
| Logger | SendToAuditLog | `bw.jms.send` | LogMessage | jmsConnection |
| InventoryCheck | StockCheck | `bw.jms.requestreply` | StockCheck | jmsConnection |
| CreateOrderProcess | SendOrderDetails | `bw.jms.send` | Orders | jmsConnection |
| AuditLog | JMSReceiveMessage | `bw.jms.receive` | LogMessage | jmsConnection |


