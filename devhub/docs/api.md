# API

Orders API
(version v1)

API for managing customer orders.

Base path: `/api`

## Endpoints

| Method | Path | Summary |
|---|---|---|
| POST | `/orders` | Place a new order. |
| GET | `/orders/{orderId}` | Retrieve details of a specific order. |
| PUT | `/orders/{orderId}` | Update the status of an existing order. |
| GET | `/orders/customer/{customerId}` | Retrieve all orders for a specific customer. |


Descriptor files:

- `OrderProcessing.module/Service Descriptors/OrdersAPI.json`

## Implemented by

Which BusinessWorks process handles each REST operation (from the module's REST service bindings).

| Method | Path | Process |
|---|---|---|
| POST | `/api/orders` | OrdersRESTInterface |
| GET | `/api/orders/{orderId}` | OrdersRESTInterface |
| PUT | `/api/orders/{orderId}` | OrdersRESTInterface |
| GET | `/api/orders/customer/{customerId}` | OrdersRESTInterface |


