# Messaging

3 destinations (queues, topics, endpoints, subjects) used by the processes. Producers include request/reply senders.

| Destination | Technology | Kind | Configuration variable | Producers | Consumers |
|---|---|---|---|---|---|
| `StockCheck` | JMS | queue | - | InventoryCheck | StockCheck |
| `LogMessage` | JMS | queue | - | Logger | AuditLog |
| `Orders` | JMS | queue | - | CreateOrderProcess | - |


EMS broker: `tcp://192.168.1.30:9011`


## Messaging activities

| Process | Activity | Technology | Type | Direction | Destination | Connection |
|---|---|---|---|---|---|---|
| StockCheck | JMSReceiveMessage | JMS | `bw.jms.receive` | receive | StockCheck | jmsConnection |
| Logger | SendToAuditLog | JMS | `bw.jms.send` | send | LogMessage | jmsConnection |
| InventoryCheck | StockCheck | JMS | `bw.jms.requestreply` | request / reply | StockCheck | jmsConnection |
| CreateOrderProcess | SendOrderDetails | JMS | `bw.jms.send` | send | Orders | jmsConnection |
| AuditLog | JMSReceiveMessage | JMS | `bw.jms.receive` | receive | LogMessage | jmsConnection |


