# Resources and configuration

## Shared resources

- **api** (`httpConnResource`) in `OrderProcessing.module/Resources/orderprocessing/module/api.httpConnResource`
- **JMSConnectionResource** (`jmsConnResource`) in `OrderProcessing.module/Resources/orderprocessing/module/JMSConnectionResource.jmsConnResource`


## Configuration variables

Values from the application's default profile, falling back to the module defaults. Names that
look like secrets, and any value that is an obfuscated password or a `%%placeholder%%`, are left
out on purpose.

| Variable | Value |
|---|---|
| `LogLevel` | `INFO` |
| `EMSURL` | `tcp://192.168.1.30:9011` |
| `EMSUserName` | `admin` |


## Systems the processes talk to

Activities other than core and general activities, with their technology and connection resource where one is named.
Values in the last column are literals typed into the activity's input mapping (host, URI, topic, table and similar).

| Process | Activity | Technology | Type | Connection | Detail |
|---|---|---|---|---|---|
| StockCheck | JMSReceiveMessage | JMS | `bw.jms.receive` | jmsConnection | destination StockCheck;  |
| StockCheck | ReplytoJMSMessage | JMS | `bw.jms.reply` | - | - |
| Logger | SendToAuditLog | JMS | `bw.jms.send` | jmsConnection | destination LogMessage;  |
| InventoryCheck | StockCheck | JMS | `bw.jms.requestreply` | jmsConnection | destination StockCheck;  |
| CreateOrderProcess | SendOrderDetails | JMS | `bw.jms.send` | jmsConnection | destination Orders;  |
| AuditLog | JMSReceiveMessage | JMS | `bw.jms.receive` | jmsConnection | destination LogMessage;  |




