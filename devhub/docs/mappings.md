# Data mappings

How data moves between systems and between activities. Every BusinessWorks activity has an input mapping (an XSLT
transformation) that builds the activity's input from the process input and from the output of earlier activities
(`$Start`, `$ActivityName`). This page lists those mappings, extracted from the process files. Expressions are shown as
written in the project, so complex ones (loops, conditions, custom functions) are shown in full rather than interpreted;
a condition such as `for each ...` applies to the rows beneath it.

## Data flow between systems

Mappings that feed an activity talking to another system (database, queue, topic, HTTP or SOAP service, file) or the
reply of a process. The source is the activity or process input the value comes from.

| Process | Source | Target | Target field | Expression |
|---|---|---|---|---|
| UpdateOrderStatus | GetSharedVariable | End (output) | `Order_status` | `$GetSharedVariable/Status[$var]` |
| StockCheck | JMSReceiveMessage (JMS) | ReplytoJMSMessage (JMS) | `Body/Product/name` | `$JMSReceiveMessage/Body/tns1:Product/tns1:name` |
| StockCheck | (computed) | ReplytoJMSMessage (JMS) | `Body/Product/quantity` | `tib:random(0, 15)` |
| QueryOrder | (computed) | End (output) | `orderId` | `tib:uuid()` |
| QueryOrder | GetCustomer | End (output) | `customerId` | `$GetCustomer/customer[$var2]/id` |
| QueryOrder | (computed) | End (output) | `orderDate` | `current-dateTime()` |
| QueryOrder | GetProduct | End (output) | `products/name` | `$GetProduct/tns1:OrderRequest_products[$var2]/tns1:name` |
| QueryOrder | (computed) | End (output) | `products/quantity` | `tib:random(1, 15)` |
| QueryOrder | GetOrderStatus | End (output) | `status` | `$GetOrderStatus/Status[$var]` |
| QueryCustomerOrders | (computed) | End (output) | `Order/orderId` | `tib:uuid()` |
| QueryCustomerOrders | GetCustomer | End (output) | `Order/customerId` | `$GetCustomer/customer[$var]/id` |
| QueryCustomerOrders | (computed) | End (output) | `Order/orderDate` | `current-dateTime()` |
| QueryCustomerOrders | GetProduct | End (output) | `Order/products/name` | `$GetProduct/tns2:OrderRequest_products[$var]/tns2:name` |
| QueryCustomerOrders | (computed) | End (output) | `Order/products/quantity` | `tib:random(1, 15)` |
| QueryCustomerOrders | GetOrderStatus | End (output) | `Order/status` | `$GetOrderStatus/Status[$var2]` |
| Logger | process input | SendToAuditLog (JMS) | `Body` | `$Start/tns1:LogMessage` |
| InventoryCheck | StockAvailable | End (output) | `InventoryAvailable` | `$StockAvailable` |
| InventoryCheck | currentElement | StockCheck (JMS) | `Body/Product` | `$currentElement/ancestor-or-self::*/namespace::node()` |
| InventoryCheck | currentElement | StockCheck (JMS) | `Body/Product` | `$currentElement/@*` |
| InventoryCheck | currentElement | StockCheck (JMS) | `Body/Product` | `$currentElement/node()` |
| DatabaseUpdate | process input | End (output) | `Order_orderId` | `$Start/tns2:orderId` |
| CreateOrderProcess | InsertOrderIntoDB | End (output) | `Order_orderId` | `$InsertOrderIntoDB` |
| CreateOrderProcess | constant | End (output) | `Order_orderId` | `""` |
| CreateOrderProcess | InsertOrderIntoDB | SendOrderDetails (JMS) | `Body/Order/orderId` | `$InsertOrderIntoDB` |
| CreateOrderProcess | process input | SendOrderDetails (JMS) | `Body/Order/customerId` | `$Start/tns1:customerId` |
| CreateOrderProcess | (computed) | SendOrderDetails (JMS) | `Body/Order/orderDate` | `current-dateTime()` |
| CreateOrderProcess | process input | SendOrderDetails (JMS) | `Body/Order` | `$Start/tns1:products` |
| CreateOrderProcess | constant | SendOrderDetails (JMS) | `Body/Order/status` | `"Processing"` |
| CreateOrderProcess | constant | SendOrderDetails (JMS) | `Body/Order/status` | `"Pending"` |


## Mappings per process


### UpdateOrderStatus


#### End

| Target field | Source expression | Takes data from |
|---|---|---|
| `Order_status` | `$GetSharedVariable/Status[$var]` | GetSharedVariable |

#### LoggerUpdateStatus

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `concat("Updating Status of Order: ",$Start)` | process input |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |

#### UpdateOrderDatabase

| Target field | Source expression | Takes data from |
|---|---|---|
| `IntervalInMillisec` | `tib:random(2,300)` |  |


### StockCheck


#### ReplytoJMSMessage

| Target field | Source expression | Takes data from |
|---|---|---|
| `Body/Product/name` | `$JMSReceiveMessage/Body/tns1:Product/tns1:name` | JMSReceiveMessage |
| `Body/Product/quantity` | `tib:random(0, 15)` |  |

#### LogCheckingStock

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `concat("Check stock levels for : ",$JMSReceiveMessage/Body/tns1:Product/tns1:name)` | JMSReceiveMessage |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |

#### LogStockLevels

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `concat("Qty in Stock for ",$JMSReceiveMessage/Body/tns1:Product/tns1:name," is ",xsd:string($JMSReceiveMessage/Body/tns1:Product/t...` | JMSReceiveMessage |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |


### QueryOrder


#### End

| Target field | Source expression | Takes data from |
|---|---|---|
| `orderId` | `tib:uuid()` |  |
| `customerId` | `$GetCustomer/customer[$var2]/id` | GetCustomer |
| `orderDate` | `current-dateTime()` |  |
| `products/name` | `$GetProduct/tns1:OrderRequest_products[$var2]/tns1:name` | GetProduct |
| `products/quantity` | `tib:random(1, 15)` |  |
| `status` | `$GetOrderStatus/Status[$var]` | GetOrderStatus |

#### Logger

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `concat("Querying Order Status for Order: ", $Start)` | process input |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |

#### OrderDatabaseQuery

| Target field | Source expression | Takes data from |
|---|---|---|
| `IntervalInMillisec` | `tib:random(2, 300)` |  |


### QueryCustomerOrders


#### End

| Target field | Source expression | Takes data from |
|---|---|---|
| `Order/orderId` | `tib:uuid()` |  |
| `Order/customerId` | `$GetCustomer/customer[$var]/id` | GetCustomer |
| `Order/orderDate` | `current-dateTime()` |  |
| `Order/products/name` | `$GetProduct/tns2:OrderRequest_products[$var]/tns2:name` | GetProduct |
| `Order/products/quantity` | `tib:random(1, 15)` |  |
| `Order/status` | `$GetOrderStatus/Status[$var2]` | GetOrderStatus |

#### Logger

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `concat("Querying Orders for Customer ID: ",$Start)` | process input |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |

#### QueryOrdersDatabase

| Target field | Source expression | Takes data from |
|---|---|---|
| `IntervalInMillisec` | `tib:random(2, 300)` |  |


### OrdersRESTInterface


#### postOut

| Target field | Source expression | Takes data from |
|---|---|---|
| `item/OrderCreationResponse/orderId` | `$CreateOrder` | CreateOrder |

#### CreateOrder

| Target field | Source expression | Takes data from |
|---|---|---|
| `customerId` | `$post.item/tns3:customerId` | post |
| `OrderRequest` | `$post.item/tns3:products` | post |

#### LogOrderPost

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `concat("Payload Received: ", tib:render-xml($post.item))` | post |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |

#### getOut

| Target field | Source expression | Takes data from |
|---|---|---|
| `item/Order/orderId` | `$get/parameters/tns4:orders1GetParameters/tns4:orderId` | get |
| `item/Order/customerId` | `$QueryOrder/tns3:customerId` | QueryOrder |
| `item/Order/orderDate` | `current-dateTime()` |  |
| `item/Order` | `$QueryOrder/tns3:products` | QueryOrder |
| `item/Order/status` | `$QueryOrder/tns3:status` | QueryOrder |

#### LogOrderGet

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `concat("Querying Order: ", tib:render-xml($get.parameters/tns4:orderId))` | get |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |

#### QueryOrder

| Target field | Source expression | Takes data from |
|---|---|---|
| `Order_orderId` | `$get.parameters/tns4:orderId` | get |

#### putOut

| Target field | Source expression | Takes data from |
|---|---|---|
| `status/statusLine/statusCode` | `200` | constant |

#### LogUpdateOrderStatus

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `concat("Updating Status for Order: ", tib:render-xml($put.parameters/tns4:orderId))` | put |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |

#### UpdateOrderStatus

| Target field | Source expression | Takes data from |
|---|---|---|
| `Order_orderId` | `$put.parameters/tns4:orderId` | put |

#### LogOrderStatusUpdated

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `concat("Order status for ",$put.parameters/tns4:orderId," was updated to: ",$UpdateOrderStatus)` | put, UpdateOrderStatus |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |

#### getOut1

| Target field | Source expression | Takes data from |
|---|---|---|
| `item/OrderArray` | `$QueryCustomerOrders/tns3:Order` | QueryCustomerOrders |

#### LogCustomerOrdersGet

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `concat("Getting Orders for Customer: ", tib:render-xml($get1.parameters/tns4:customerId))` | get1 |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |

#### QueryCustomerOrders

| Target field | Source expression | Takes data from |
|---|---|---|
| `OrderRequest_customerId` | `$get1.parameters/tns4:customerId` | get1 |


### Logger


#### Log

| Target field | Source expression | Takes data from |
|---|---|---|
| `logLevel` | `$Start/tns1:Severity` | process input |
| `message` | `$Start/tns1:LogMessage` | process input |

#### SendToAuditLog

| Target field | Source expression | Takes data from |
|---|---|---|
| `Body` | `$Start/tns1:LogMessage` | process input |


### InventoryCheck


#### End

| Target field | Source expression | Takes data from |
|---|---|---|
| `InventoryAvailable` | `$StockAvailable` | StockAvailable |

#### Logger

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `concat("Checking Inventory for Order: ",$Start/tns1:orderId)` | process input |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |

#### StockCheck

| Target field | Source expression | Takes data from |
|---|---|---|
| `Body/Product` | `$currentElement/ancestor-or-self::*/namespace::node()` | currentElement |
| `Body/Product` | `$currentElement/@*` | currentElement |
| `Body/Product` | `$currentElement/node()` | currentElement |


### DatabaseUpdate


#### End

| Target field | Source expression | Takes data from |
|---|---|---|
| `Order_orderId` | `$Start/tns2:orderId` | process input |

#### Logger

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `"Updating Database"` | constant |
| `Severity` | `"INFO"` | constant |

#### DatabaseUpdate

| Target field | Source expression | Takes data from |
|---|---|---|
| `IntervalInMillisec` | `tib:random(2, 300)` |  |


### CreateOrderProcess


#### End

| Target field | Source expression | Takes data from |
|---|---|---|
| `Order_orderId` | `$InsertOrderIntoDB` (when $InventoryCheck/tns3:InventoryAvailable = true()) | InsertOrderIntoDB |
| `Order_orderId` | `""` (otherwise) | constant |

#### Logger

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `"Creating Order"` | constant |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |

#### InsertOrderIntoDB

| Target field | Source expression | Takes data from |
|---|---|---|
| `orderId` | `tib:uuid()` |  |
| `customerId` | `$Start/tns1:customerId` | process input |
| `orderDate` | `current-dateTime()` |  |
| `Order` | `$Start/tns1:products` | process input |
| `status` | `"Pending"` | constant |

#### InventoryCheck

| Target field | Source expression | Takes data from |
|---|---|---|
| `orderId` | `$InsertOrderIntoDB` | InsertOrderIntoDB |
| `customerId` | `$Start/tns1:customerId` | process input |
| `orderDate` | `current-dateTime()` |  |
| `Order` | `$Start/tns1:products` | process input |
| `status` | `"Pending"` | constant |

#### SendOrderDetails

| Target field | Source expression | Takes data from |
|---|---|---|
| `Body/Order/orderId` | `$InsertOrderIntoDB` | InsertOrderIntoDB |
| `Body/Order/customerId` | `$Start/tns1:customerId` | process input |
| `Body/Order/orderDate` | `current-dateTime()` |  |
| `Body/Order` | `$Start/tns1:products` | process input |
| `Body/Order/status` | `"Processing"` (when $InventoryCheck/tns3:InventoryAvailable = true()) | constant |
| `Body/Order/status` | `"Pending"` (otherwise) | constant |

#### OrderCreatedLog

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `concat("Created Order: ",$InsertOrderIntoDB)` | InsertOrderIntoDB |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |

#### NoStockAvailable

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `"No Stock Available to fulfill this order"` | constant |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |


### AuditLog


#### UpdateAuditDatabase

| Target field | Source expression | Takes data from |
|---|---|---|
| `IntervalInMillisec` | `tib:random(2, 300)` |  |


### Activator


#### StartupLogger

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `"Order Processing Application Started"` | constant |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |

#### SetupOrderStatusVariable

| Target field | Source expression | Takes data from |
|---|---|---|
| `Status` | `"Pending"` | constant |
| `Status` | `"Processing"` | constant |
| `Status` | `"Shipped"` | constant |
| `Status` | `"Cancelled"` | constant |

#### SetupCustomersVariable

| Target field | Source expression | Takes data from |
|---|---|---|
| `customer/name` | `"Apexian Cognition Labs"` | constant |
| `customer/id` | `"1001"` | constant |
| `customer/name` | `"Quantum Temporal Dynamics Corp."` | constant |
| `customer/id` | `"1002"` | constant |
| `customer/name` | `"Aetheric Signal Solutions Inc."` | constant |
| `customer/id` | `"1003"` | constant |
| `customer/name` | `"Resonance AI Analytics"` | constant |
| `customer/id` | `"1005"` | constant |
| `customer/name` | `"Chronosync Technologies Ltd."` | constant |
| `customer/id` | `"1006"` | constant |

#### SetupProductsVariable

| Target field | Source expression | Takes data from |
|---|---|---|
| `OrderRequest_products/name` | `"ChronoSyncular Quantum Harmonizer"` | constant |
| `OrderRequest_products/quantity` | `5` | constant |
| `OrderRequest_products/name` | `"Aetherium Vectorial Transducer"` | constant |
| `OrderRequest_products/quantity` | `5` | constant |
| `OrderRequest_products/name` | `"Stochastic Resonance Modulator Array"` | constant |
| `OrderRequest_products/quantity` | `5` | constant |
| `OrderRequest_products/name` | `"Polyphasic Temporal Displacement Engine"` | constant |
| `OrderRequest_products/quantity` | `5` | constant |
| `OrderRequest_products/name` | `"Cognitive-Emotive Resonance Cascade"` | constant |
| `OrderRequest_products/quantity` | `5` | constant |

#### ShutdownLogger

| Target field | Source expression | Takes data from |
|---|---|---|
| `LogMessage` | `"Shutting Down Order Processing Application"` | constant |
| `Severity` | `bw:getModuleProperty("LogLevel")` |  ; config LogLevel |


