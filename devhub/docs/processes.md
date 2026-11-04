# Processes

11 processes with 23 palette activities (calls to other processes are listed under *Calls*).

## Inventory

| Process | Triggered by | Activities | Calls | Fault handlers |
|---|---|---|---|---|
| UpdateOrderStatus | Sub-process, called by OrdersRESTInterface | 2 | Logger | 0 |
| StockCheck | JMS StockCheck | 3 | Logger | 0 |
| QueryOrder | Sub-process, called by OrdersRESTInterface | 2 | Logger | 0 |
| QueryCustomerOrders | Sub-process, called by OrdersRESTInterface | 2 | Logger | 0 |
| OrdersRESTInterface | REST POST /api/orders, GET /api/orders/{orderId}, PUT /api/orders/{orderId}, GET /api/orders/customer/{customerId} | 0 | CreateOrderProcess, Logger, QueryOrder, UpdateOrderStatus, QueryCustomerOrders | 0 |
| Logger | Sub-process, called by UpdateOrderStatus, StockCheck, QueryOrder, QueryCustomerOrders, OrdersRESTInterface, InventoryCheck, DatabaseUpdate, CreateOrderProcess, Activator | 3 | - | 0 |
| InventoryCheck | Sub-process, called by CreateOrderProcess | 5 | Logger | 0 |
| DatabaseUpdate | Sub-process, called by CreateOrderProcess | 2 | Logger | 0 |
| CreateOrderProcess | Sub-process, called by OrdersRESTInterface | 2 | Logger, DatabaseUpdate, InventoryCheck | 0 |
| AuditLog | JMS LogMessage | 2 | - | 0 |
| Activator | Module start/stop | 0 | Logger | 0 |


## Call tree

Each entry point (REST operation, JMS receiver, timer, module activator) and the processes it calls, in call order.

- **StockCheck** (JMS StockCheck)
    - **Logger**
- **OrdersRESTInterface** (REST POST /api/orders, GET /api/orders/{orderId}, PUT /api/orders/{orderId}, GET /api/orders/customer/{customerId})
    - **CreateOrderProcess**
        - **Logger**
        - **DatabaseUpdate**
            - **Logger**
        - **InventoryCheck**
            - **Logger**
    - **Logger**
    - **QueryOrder**
        - **Logger**
    - **UpdateOrderStatus**
        - **Logger**
    - **QueryCustomerOrders**
        - **Logger**
- **AuditLog** (JMS LogMessage)
- **Activator** (Module start/stop)
    - **Logger**


## Process details

### UpdateOrderStatus

- File: `OrderProcessing.module/Processes/orderprocessing/module/UpdateOrderStatus.bwp`
- Triggered by: Sub-process, called by OrdersRESTInterface
- Called by: OrdersRESTInterface
- Activities:
    - `bw.internal.end` x 1
    - `bw.generalactivities.sleep` x 1
- Flow: see below.


#### Flow of UpdateOrderStatus

<div style="overflow-x:auto"><svg xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Flow of UpdateOrderStatus" width="1050" height="66" viewBox="0 0 1050 66" font-family="Helvetica,Arial,sans-serif"><defs><marker id="ah-1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#6b7c93"/></marker></defs><path d="M180,33 C202,33 203,33 225,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-1)"/><path d="M395,33 C417,33 418,33 440,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-1)"/><path d="M610,33 C632,33 633,33 655,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-1)"/><path d="M825,33 C847,33 848,33 870,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-1)"/><g><title>Start</title><rect x="10" y="10" width="170" height="46" rx="8" fill="#c8e6c9" stroke="#6b7c93"/><text x="95" y="38" text-anchor="middle" font-size="12" fill="#1f2933">Start</text></g><g><title>LoggerUpdateStatus</title><rect x="225" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="310" y="38" text-anchor="middle" font-size="12" fill="#1f2933">LoggerUpdateStatus</text></g><g><title>GetSharedVariable</title><rect x="440" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="525" y="38" text-anchor="middle" font-size="12" fill="#1f2933">GetSharedVariable</text></g><g><title>UpdateOrderDatabase</title><rect x="655" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="740" y="29" text-anchor="middle" font-size="12" fill="#1f2933">UpdateOrderDatabase</text><text x="740" y="45" text-anchor="middle" font-size="9" fill="#52606d">generalactivities.sleep</text></g><g><title>End</title><rect x="870" y="10" width="170" height="46" rx="8" fill="#ffe0b2" stroke="#6b7c93"/><text x="955" y="29" text-anchor="middle" font-size="12" fill="#1f2933">End</text><text x="955" y="45" text-anchor="middle" font-size="9" fill="#52606d">internal.end</text></g></svg></div>

| From | To |
|---|---|
| Start | LoggerUpdateStatus |
| LoggerUpdateStatus | GetSharedVariable |
| GetSharedVariable | UpdateOrderDatabase |
| UpdateOrderDatabase | End |


### StockCheck

- File: `OrderProcessing.module/Processes/orderprocessing/module/StockCheck.bwp`
- Triggered by: JMS StockCheck
- Activities:
    - `bw.jms.receive` x 1
    - `bw.jms.reply` x 1
    - `bw.generalactivities.confirm` x 1
- Flow: see below.
- Activities that talk to other systems:
    - **JMSReceiveMessage** (`bw.jms.receive`): destination StockCheck, connection `jmsConnection`
    - **ReplytoJMSMessage** (`bw.jms.reply`)


#### Flow of StockCheck

<div style="overflow-x:auto"><svg xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Flow of StockCheck" width="1050" height="66" viewBox="0 0 1050 66" font-family="Helvetica,Arial,sans-serif"><defs><marker id="ah-2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#6b7c93"/></marker></defs><path d="M180,33 C202,33 203,33 225,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-2)"/><path d="M610,33 C632,33 633,33 655,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-2)"/><path d="M825,33 C847,33 848,33 870,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-2)"/><path d="M395,33 C417,33 418,33 440,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-2)"/><g><title>JMSReceiveMessage</title><rect x="10" y="10" width="170" height="46" rx="8" fill="#c8e6c9" stroke="#6b7c93"/><text x="95" y="29" text-anchor="middle" font-size="12" fill="#1f2933">JMSReceiveMessage</text><text x="95" y="45" text-anchor="middle" font-size="9" fill="#52606d">jms.receive</text></g><g><title>LogCheckingStock</title><rect x="225" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="310" y="38" text-anchor="middle" font-size="12" fill="#1f2933">LogCheckingStock</text></g><g><title>ReplytoJMSMessage</title><rect x="440" y="10" width="170" height="46" rx="8" fill="#ffe0b2" stroke="#6b7c93"/><text x="525" y="29" text-anchor="middle" font-size="12" fill="#1f2933">ReplytoJMSMessage</text><text x="525" y="45" text-anchor="middle" font-size="9" fill="#52606d">jms.reply</text></g><g><title>Confirm</title><rect x="655" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="740" y="29" text-anchor="middle" font-size="12" fill="#1f2933">Confirm</text><text x="740" y="45" text-anchor="middle" font-size="9" fill="#52606d">generalactivities.confirm</text></g><g><title>LogStockLevels</title><rect x="870" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="955" y="38" text-anchor="middle" font-size="12" fill="#1f2933">LogStockLevels</text></g></svg></div>

| From | To |
|---|---|
| JMSReceiveMessage | LogCheckingStock |
| ReplytoJMSMessage | Confirm |
| Confirm | LogStockLevels |
| LogCheckingStock | ReplytoJMSMessage |


### QueryOrder

- File: `OrderProcessing.module/Processes/orderprocessing/module/QueryOrder.bwp`
- Triggered by: Sub-process, called by OrdersRESTInterface
- Called by: OrdersRESTInterface
- Activities:
    - `bw.internal.end` x 1
    - `bw.generalactivities.sleep` x 1
- Flow: see below.


#### Flow of QueryOrder

<div style="overflow-x:auto"><svg xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Flow of QueryOrder" width="1480" height="66" viewBox="0 0 1480 66" font-family="Helvetica,Arial,sans-serif"><defs><marker id="ah-3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#6b7c93"/></marker></defs><path d="M180,33 C202,33 203,33 225,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-3)"/><path d="M395,33 C417,33 418,33 440,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-3)"/><path d="M610,33 C632,33 633,33 655,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-3)"/><path d="M825,33 C847,33 848,33 870,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-3)"/><path d="M1040,33 C1062,33 1063,33 1085,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-3)"/><path d="M1255,33 C1277,33 1278,33 1300,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-3)"/><g><title>Start</title><rect x="10" y="10" width="170" height="46" rx="8" fill="#c8e6c9" stroke="#6b7c93"/><text x="95" y="38" text-anchor="middle" font-size="12" fill="#1f2933">Start</text></g><g><title>Logger</title><rect x="225" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="310" y="38" text-anchor="middle" font-size="12" fill="#1f2933">Logger</text></g><g><title>OrderDatabaseQuery</title><rect x="440" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="525" y="29" text-anchor="middle" font-size="12" fill="#1f2933">OrderDatabaseQuery</text><text x="525" y="45" text-anchor="middle" font-size="9" fill="#52606d">generalactivities.sleep</text></g><g><title>GetOrderStatus</title><rect x="655" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="740" y="38" text-anchor="middle" font-size="12" fill="#1f2933">GetOrderStatus</text></g><g><title>GetCustomer</title><rect x="870" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="955" y="38" text-anchor="middle" font-size="12" fill="#1f2933">GetCustomer</text></g><g><title>GetProduct</title><rect x="1085" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="1170" y="38" text-anchor="middle" font-size="12" fill="#1f2933">GetProduct</text></g><g><title>End</title><rect x="1300" y="10" width="170" height="46" rx="8" fill="#ffe0b2" stroke="#6b7c93"/><text x="1385" y="29" text-anchor="middle" font-size="12" fill="#1f2933">End</text><text x="1385" y="45" text-anchor="middle" font-size="9" fill="#52606d">internal.end</text></g></svg></div>

| From | To |
|---|---|
| Start | Logger |
| Logger | OrderDatabaseQuery |
| OrderDatabaseQuery | GetOrderStatus |
| GetOrderStatus | GetCustomer |
| GetCustomer | GetProduct |
| GetProduct | End |


### QueryCustomerOrders

- File: `OrderProcessing.module/Processes/orderprocessing/module/QueryCustomerOrders.bwp`
- Triggered by: Sub-process, called by OrdersRESTInterface
- Called by: OrdersRESTInterface
- Activities:
    - `bw.internal.end` x 1
    - `bw.generalactivities.sleep` x 1
- Flow: see below.


#### Flow of QueryCustomerOrders

<div style="overflow-x:auto"><svg xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Flow of QueryCustomerOrders" width="1480" height="66" viewBox="0 0 1480 66" font-family="Helvetica,Arial,sans-serif"><defs><marker id="ah-4" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#6b7c93"/></marker></defs><path d="M180,33 C202,33 203,33 225,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-4)"/><path d="M395,33 C417,33 418,33 440,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-4)"/><path d="M610,33 C632,33 633,33 655,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-4)"/><path d="M825,33 C847,33 848,33 870,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-4)"/><path d="M1040,33 C1062,33 1063,33 1085,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-4)"/><path d="M1255,33 C1277,33 1278,33 1300,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-4)"/><g><title>Start</title><rect x="10" y="10" width="170" height="46" rx="8" fill="#c8e6c9" stroke="#6b7c93"/><text x="95" y="38" text-anchor="middle" font-size="12" fill="#1f2933">Start</text></g><g><title>Logger</title><rect x="225" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="310" y="38" text-anchor="middle" font-size="12" fill="#1f2933">Logger</text></g><g><title>QueryOrdersDatabase</title><rect x="440" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="525" y="29" text-anchor="middle" font-size="12" fill="#1f2933">QueryOrdersDatabase</text><text x="525" y="45" text-anchor="middle" font-size="9" fill="#52606d">generalactivities.sleep</text></g><g><title>GetCustomer</title><rect x="655" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="740" y="38" text-anchor="middle" font-size="12" fill="#1f2933">GetCustomer</text></g><g><title>GetProduct</title><rect x="870" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="955" y="38" text-anchor="middle" font-size="12" fill="#1f2933">GetProduct</text></g><g><title>GetOrderStatus</title><rect x="1085" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="1170" y="38" text-anchor="middle" font-size="12" fill="#1f2933">GetOrderStatus</text></g><g><title>End</title><rect x="1300" y="10" width="170" height="46" rx="8" fill="#ffe0b2" stroke="#6b7c93"/><text x="1385" y="29" text-anchor="middle" font-size="12" fill="#1f2933">End</text><text x="1385" y="45" text-anchor="middle" font-size="9" fill="#52606d">internal.end</text></g></svg></div>

| From | To |
|---|---|
| Start | Logger |
| Logger | QueryOrdersDatabase |
| QueryOrdersDatabase | GetCustomer |
| GetCustomer | GetProduct |
| GetProduct | GetOrderStatus |
| GetOrderStatus | End |


### OrdersRESTInterface

- File: `OrderProcessing.module/Processes/orderprocessing/module/OrdersRESTInterface.bwp`
- Triggered by: REST POST /api/orders, GET /api/orders/{orderId}, PUT /api/orders/{orderId}, GET /api/orders/customer/{customerId}
- Flow: see below.


#### Flow of OrdersRESTInterface

<div style="overflow-x:auto"><svg xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Flow of OrdersRESTInterface" width="835" height="264" viewBox="0 0 835 264" font-family="Helvetica,Arial,sans-serif"><defs><marker id="ah-5" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#6b7c93"/></marker></defs><path d="M395,33 C417,33 418,33 440,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-5)"/><path d="M180,33 C202,33 203,33 225,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-5)"/><path d="M180,99 C202,99 203,99 225,99" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-5)"/><path d="M395,99 C417,99 418,99 440,99" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-5)"/><path d="M180,165 C202,165 203,165 225,165" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-5)"/><path d="M395,165 C417,165 418,165 440,165" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-5)"/><path d="M610,165 C632,165 633,33 655,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-5)"/><path d="M180,231 C202,231 203,231 225,231" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-5)"/><path d="M395,231 C417,231 418,231 440,231" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-5)"/><g><title>CreateOrder</title><rect x="225" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="310" y="38" text-anchor="middle" font-size="12" fill="#1f2933">CreateOrder</text></g><g><title>postOut</title><rect x="440" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="525" y="38" text-anchor="middle" font-size="12" fill="#1f2933">postOut</text></g><g><title>LogOrderPost</title><rect x="10" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="95" y="38" text-anchor="middle" font-size="12" fill="#1f2933">LogOrderPost</text></g><g><title>LogOrderGet</title><rect x="10" y="76" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="95" y="104" text-anchor="middle" font-size="12" fill="#1f2933">LogOrderGet</text></g><g><title>QueryOrder</title><rect x="225" y="76" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="310" y="104" text-anchor="middle" font-size="12" fill="#1f2933">QueryOrder</text></g><g><title>getOut</title><rect x="440" y="76" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="525" y="104" text-anchor="middle" font-size="12" fill="#1f2933">getOut</text></g><g><title>LogUpdateOrderStatus</title><rect x="10" y="142" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="95" y="170" text-anchor="middle" font-size="12" fill="#1f2933">LogUpdateOrderStatus</text></g><g><title>UpdateOrderStatus</title><rect x="225" y="142" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="310" y="170" text-anchor="middle" font-size="12" fill="#1f2933">UpdateOrderStatus</text></g><g><title>LogOrderStatusUpdated</title><rect x="440" y="142" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="525" y="170" text-anchor="middle" font-size="12" fill="#1f2933">LogOrderStatusUpdated</text></g><g><title>putOut</title><rect x="655" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="740" y="38" text-anchor="middle" font-size="12" fill="#1f2933">putOut</text></g><g><title>LogCustomerOrdersGet</title><rect x="10" y="208" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="95" y="236" text-anchor="middle" font-size="12" fill="#1f2933">LogCustomerOrdersGet</text></g><g><title>QueryCustomerOrders</title><rect x="225" y="208" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="310" y="236" text-anchor="middle" font-size="12" fill="#1f2933">QueryCustomerOrders</text></g><g><title>getOut1</title><rect x="440" y="208" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="525" y="236" text-anchor="middle" font-size="12" fill="#1f2933">getOut1</text></g></svg></div>

| From | To |
|---|---|
| CreateOrder | postOut |
| LogOrderPost | CreateOrder |
| LogOrderGet | QueryOrder |
| QueryOrder | getOut |
| LogUpdateOrderStatus | UpdateOrderStatus |
| UpdateOrderStatus | LogOrderStatusUpdated |
| LogOrderStatusUpdated | putOut |
| LogCustomerOrdersGet | QueryCustomerOrders |
| QueryCustomerOrders | getOut1 |


### Logger

- File: `OrderProcessing.module/Processes/orderprocessing/module/Logger.bwp`
- Triggered by: Sub-process, called by UpdateOrderStatus, StockCheck, QueryOrder, QueryCustomerOrders, OrdersRESTInterface, InventoryCheck, DatabaseUpdate, CreateOrderProcess, Activator
- Called by: UpdateOrderStatus, StockCheck, QueryOrder, QueryCustomerOrders, OrdersRESTInterface, InventoryCheck, DatabaseUpdate, CreateOrderProcess, Activator
- Activities:
    - `bw.internal.end` x 1
    - `bw.generalactivities.log` x 1
    - `bw.jms.send` x 1
- Flow: see below.
- Activities that talk to other systems:
    - **SendToAuditLog** (`bw.jms.send`): destination LogMessage, connection `jmsConnection`


#### Flow of Logger

<div style="overflow-x:auto"><svg xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Flow of Logger" width="835" height="66" viewBox="0 0 835 66" font-family="Helvetica,Arial,sans-serif"><defs><marker id="ah-6" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#6b7c93"/></marker></defs><path d="M180,33 C202,33 203,33 225,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-6)"/><path d="M395,33 C417,33 418,33 440,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-6)"/><path d="M610,33 C632,33 633,33 655,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-6)"/><g><title>Start</title><rect x="10" y="10" width="170" height="46" rx="8" fill="#c8e6c9" stroke="#6b7c93"/><text x="95" y="38" text-anchor="middle" font-size="12" fill="#1f2933">Start</text></g><g><title>Log</title><rect x="225" y="10" width="170" height="46" rx="8" fill="#eceff1" stroke="#6b7c93"/><text x="310" y="29" text-anchor="middle" font-size="12" fill="#1f2933">Log</text><text x="310" y="45" text-anchor="middle" font-size="9" fill="#52606d">generalactivities.log</text></g><g><title>SendToAuditLog</title><rect x="440" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="525" y="29" text-anchor="middle" font-size="12" fill="#1f2933">SendToAuditLog</text><text x="525" y="45" text-anchor="middle" font-size="9" fill="#52606d">jms.send</text></g><g><title>End</title><rect x="655" y="10" width="170" height="46" rx="8" fill="#ffe0b2" stroke="#6b7c93"/><text x="740" y="29" text-anchor="middle" font-size="12" fill="#1f2933">End</text><text x="740" y="45" text-anchor="middle" font-size="9" fill="#52606d">internal.end</text></g></svg></div>

| From | To |
|---|---|
| Start | Log |
| Log | SendToAuditLog |
| SendToAuditLog | End |


### InventoryCheck

- File: `OrderProcessing.module/Processes/orderprocessing/module/InventoryCheck.bwp`
- Triggered by: Sub-process, called by CreateOrderProcess
- Called by: CreateOrderProcess
- Activities:
    - `bw.internal.end` x 1
    - `bw.jms.requestreply` x 1
    - `bw.generalactivities.bwassign` x 3
- Flow: see below.
- Activities that talk to other systems:
    - **StockCheck** (`bw.jms.requestreply`): destination StockCheck, connection `jmsConnection`


#### Flow of InventoryCheck

<div style="overflow-x:auto"><svg xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Flow of InventoryCheck" width="1050" height="198" viewBox="0 0 1050 198" font-family="Helvetica,Arial,sans-serif"><defs><marker id="ah-7" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#6b7c93"/></marker></defs><path d="M180,33 C202,33 203,33 225,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-7)"/><path d="M395,33 C417,33 418,33 440,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-7)"/><path d="M825,33 C847,33 848,33 870,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-7)"/><path d="M180,99 C202,99 203,99 225,99" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-7)"/><path d="M395,99 C417,99 418,99 440,99" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-7)"/><path d="M395,99 C417,99 418,165 440,165" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-7)"/><path d="M610,99 C632,99 633,99 655,99" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-7)"/><path d="M610,165 C632,165 633,99 655,99" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-7)"/><path d="M610,33 C632,33 633,33 655,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-7)"/><g><title>Start</title><rect x="10" y="10" width="170" height="46" rx="8" fill="#c8e6c9" stroke="#6b7c93"/><text x="95" y="38" text-anchor="middle" font-size="12" fill="#1f2933">Start</text></g><g><title>Logger</title><rect x="225" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="310" y="38" text-anchor="middle" font-size="12" fill="#1f2933">Logger</text></g><g><title>SetInitialAvailability</title><rect x="440" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="525" y="29" text-anchor="middle" font-size="12" fill="#1f2933">SetInitialAvailability</text><text x="525" y="45" text-anchor="middle" font-size="9" fill="#52606d">generalactivities.bwassign</text></g><g><title>Iterate</title><rect x="655" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="740" y="38" text-anchor="middle" font-size="12" fill="#1f2933">Iterate</text></g><g><title>End</title><rect x="870" y="10" width="170" height="46" rx="8" fill="#ffe0b2" stroke="#6b7c93"/><text x="955" y="29" text-anchor="middle" font-size="12" fill="#1f2933">End</text><text x="955" y="45" text-anchor="middle" font-size="9" fill="#52606d">internal.end</text></g><g><title>GroupStart</title><rect x="10" y="76" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="95" y="104" text-anchor="middle" font-size="12" fill="#1f2933">GroupStart</text></g><g><title>StockCheck</title><rect x="225" y="76" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="310" y="95" text-anchor="middle" font-size="12" fill="#1f2933">StockCheck</text><text x="310" y="111" text-anchor="middle" font-size="9" fill="#52606d">jms.requestreply</text></g><g><title>SomeStockAvailable</title><rect x="440" y="76" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="525" y="95" text-anchor="middle" font-size="12" fill="#1f2933">SomeStockAvailable</text><text x="525" y="111" text-anchor="middle" font-size="9" fill="#52606d">generalactivities.bwassign</text></g><g><title>SomeStockNotAvailable</title><rect x="440" y="142" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="525" y="161" text-anchor="middle" font-size="12" fill="#1f2933">SomeStockNotAvailable</text><text x="525" y="177" text-anchor="middle" font-size="9" fill="#52606d">generalactivities.bwassign</text></g><g><title>GroupEnd</title><rect x="655" y="76" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="740" y="104" text-anchor="middle" font-size="12" fill="#1f2933">GroupEnd</text></g></svg></div>

| From | To |
|---|---|
| Start | Logger |
| Logger | SetInitialAvailability |
| Iterate | End |
| GroupStart | StockCheck |
| StockCheck | SomeStockAvailable |
| StockCheck | SomeStockNotAvailable |
| SomeStockAvailable | GroupEnd |
| SomeStockNotAvailable | GroupEnd |
| SetInitialAvailability | Iterate |


### DatabaseUpdate

- File: `OrderProcessing.module/Processes/orderprocessing/module/DatabaseUpdate.bwp`
- Triggered by: Sub-process, called by CreateOrderProcess
- Called by: CreateOrderProcess
- Activities:
    - `bw.internal.end` x 1
    - `bw.generalactivities.sleep` x 1
- Flow: see below.


#### Flow of DatabaseUpdate

<div style="overflow-x:auto"><svg xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Flow of DatabaseUpdate" width="835" height="66" viewBox="0 0 835 66" font-family="Helvetica,Arial,sans-serif"><defs><marker id="ah-8" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#6b7c93"/></marker></defs><path d="M180,33 C202,33 203,33 225,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-8)"/><path d="M395,33 C417,33 418,33 440,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-8)"/><path d="M610,33 C632,33 633,33 655,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-8)"/><g><title>Start</title><rect x="10" y="10" width="170" height="46" rx="8" fill="#c8e6c9" stroke="#6b7c93"/><text x="95" y="38" text-anchor="middle" font-size="12" fill="#1f2933">Start</text></g><g><title>Logger</title><rect x="225" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="310" y="38" text-anchor="middle" font-size="12" fill="#1f2933">Logger</text></g><g><title>DatabaseUpdate</title><rect x="440" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="525" y="29" text-anchor="middle" font-size="12" fill="#1f2933">DatabaseUpdate</text><text x="525" y="45" text-anchor="middle" font-size="9" fill="#52606d">generalactivities.sleep</text></g><g><title>End</title><rect x="655" y="10" width="170" height="46" rx="8" fill="#ffe0b2" stroke="#6b7c93"/><text x="740" y="29" text-anchor="middle" font-size="12" fill="#1f2933">End</text><text x="740" y="45" text-anchor="middle" font-size="9" fill="#52606d">internal.end</text></g></svg></div>

| From | To |
|---|---|
| Start | Logger |
| Logger | DatabaseUpdate |
| DatabaseUpdate | End |


### CreateOrderProcess

- File: `OrderProcessing.module/Processes/orderprocessing/module/CreateOrderProcess.bwp`
- Triggered by: Sub-process, called by OrdersRESTInterface
- Called by: OrdersRESTInterface
- Activities:
    - `bw.internal.end` x 1
    - `bw.jms.send` x 1
- Flow: see below.
- Activities that talk to other systems:
    - **SendOrderDetails** (`bw.jms.send`): destination Orders, connection `jmsConnection`


#### Flow of CreateOrderProcess

<div style="overflow-x:auto"><svg xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Flow of CreateOrderProcess" width="1695" height="132" viewBox="0 0 1695 132" font-family="Helvetica,Arial,sans-serif"><defs><marker id="ah-9" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#6b7c93"/></marker></defs><path d="M180,33 C202,33 203,33 225,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-9)"/><path d="M395,33 C417,33 418,33 440,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-9)"/><path d="M610,33 C632,33 633,33 655,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-9)"/><path d="M825,33 C847,33 848,33 870,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-9)"/><path d="M825,33 C847,33 848,99 870,99" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-9)"/><path d="M1040,33 C1062,33 1063,33 1085,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-9)"/><path d="M1255,33 C1277,33 1278,33 1300,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-9)"/><path d="M1040,99 C1062,99 1278,33 1300,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-9)"/><path d="M1470,33 C1492,33 1493,33 1515,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-9)"/><g><title>Start</title><rect x="10" y="10" width="170" height="46" rx="8" fill="#c8e6c9" stroke="#6b7c93"/><text x="95" y="38" text-anchor="middle" font-size="12" fill="#1f2933">Start</text></g><g><title>Logger</title><rect x="225" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="310" y="38" text-anchor="middle" font-size="12" fill="#1f2933">Logger</text></g><g><title>InsertOrderIntoDB</title><rect x="440" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="525" y="38" text-anchor="middle" font-size="12" fill="#1f2933">InsertOrderIntoDB</text></g><g><title>InventoryCheck</title><rect x="655" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="740" y="38" text-anchor="middle" font-size="12" fill="#1f2933">InventoryCheck</text></g><g><title>SendOrderDetails</title><rect x="870" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="955" y="29" text-anchor="middle" font-size="12" fill="#1f2933">SendOrderDetails</text><text x="955" y="45" text-anchor="middle" font-size="9" fill="#52606d">jms.send</text></g><g><title>NoStockAvailable</title><rect x="870" y="76" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="955" y="104" text-anchor="middle" font-size="12" fill="#1f2933">NoStockAvailable</text></g><g><title>OrderCreatedLog</title><rect x="1085" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="1170" y="38" text-anchor="middle" font-size="12" fill="#1f2933">OrderCreatedLog</text></g><g><title>Empty</title><rect x="1300" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="1385" y="38" text-anchor="middle" font-size="12" fill="#1f2933">Empty</text></g><g><title>End</title><rect x="1515" y="10" width="170" height="46" rx="8" fill="#ffe0b2" stroke="#6b7c93"/><text x="1600" y="29" text-anchor="middle" font-size="12" fill="#1f2933">End</text><text x="1600" y="45" text-anchor="middle" font-size="9" fill="#52606d">internal.end</text></g></svg></div>

| From | To |
|---|---|
| Start | Logger |
| Logger | InsertOrderIntoDB |
| InsertOrderIntoDB | InventoryCheck |
| InventoryCheck | SendOrderDetails |
| InventoryCheck | NoStockAvailable |
| SendOrderDetails | OrderCreatedLog |
| OrderCreatedLog | Empty |
| NoStockAvailable | Empty |
| Empty | End |


### AuditLog

- File: `OrderProcessing.module/Processes/orderprocessing/module/AuditLog.bwp`
- Triggered by: JMS LogMessage
- Activities:
    - `bw.jms.receive` x 1
    - `bw.generalactivities.sleep` x 1
- Flow: see below.
- Activities that talk to other systems:
    - **JMSReceiveMessage** (`bw.jms.receive`): destination LogMessage, connection `jmsConnection`


#### Flow of AuditLog

<div style="overflow-x:auto"><svg xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Flow of AuditLog" width="405" height="66" viewBox="0 0 405 66" font-family="Helvetica,Arial,sans-serif"><defs><marker id="ah-10" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#6b7c93"/></marker></defs><path d="M180,33 C202,33 203,33 225,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-10)"/><g><title>JMSReceiveMessage</title><rect x="10" y="10" width="170" height="46" rx="8" fill="#c8e6c9" stroke="#6b7c93"/><text x="95" y="29" text-anchor="middle" font-size="12" fill="#1f2933">JMSReceiveMessage</text><text x="95" y="45" text-anchor="middle" font-size="9" fill="#52606d">jms.receive</text></g><g><title>UpdateAuditDatabase</title><rect x="225" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="310" y="29" text-anchor="middle" font-size="12" fill="#1f2933">UpdateAuditDatabase</text><text x="310" y="45" text-anchor="middle" font-size="9" fill="#52606d">generalactivities.sleep</text></g></svg></div>

| From | To |
|---|---|
| JMSReceiveMessage | UpdateAuditDatabase |


### Activator

- File: `OrderProcessing.module/Processes/orderprocessing/module/Activator.bwp`
- Triggered by: Module start/stop
- Flow: see below.


#### Flow of Activator

<div style="overflow-x:auto"><svg xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Flow of Activator" width="835" height="66" viewBox="0 0 835 66" font-family="Helvetica,Arial,sans-serif"><defs><marker id="ah-11" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#6b7c93"/></marker></defs><path d="M180,33 C202,33 203,33 225,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-11)"/><path d="M395,33 C417,33 418,33 440,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-11)"/><path d="M610,33 C632,33 633,33 655,33" fill="none" stroke="#6b7c93" stroke-width="1.3" marker-end="url(#ah-11)"/><g><title>StartupLogger</title><rect x="10" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="95" y="38" text-anchor="middle" font-size="12" fill="#1f2933">StartupLogger</text></g><g><title>SetupOrderStatusVariable</title><rect x="225" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="310" y="38" text-anchor="middle" font-size="12" fill="#1f2933">SetupOrderStatusVariable</text></g><g><title>SetupCustomersVariable</title><rect x="440" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="525" y="38" text-anchor="middle" font-size="12" fill="#1f2933">SetupCustomersVariable</text></g><g><title>SetupProductsVariable</title><rect x="655" y="10" width="170" height="46" rx="8" fill="#e8f4f8" stroke="#6b7c93"/><text x="740" y="38" text-anchor="middle" font-size="12" fill="#1f2933">SetupProductsVariable</text></g></svg></div>

| From | To |
|---|---|
| StartupLogger | SetupOrderStatusVariable |
| SetupOrderStatusVariable | SetupCustomersVariable |
| SetupCustomersVariable | SetupProductsVariable |



