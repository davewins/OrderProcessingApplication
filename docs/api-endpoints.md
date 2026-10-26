# API Endpoints

This document describes the API provided by the OrderProcessing application, based on its Swagger definition.

## OrderProcessing API

This API provides operations for managing customer orders.

### Base URL

`https://bwce.vewins.co.uk:443/tibco/apps/d0mrfgqd9jcaeh9qlih0/Orders`

### Schemes

`https`

### Consumes

`application/json`

### Produces

`application/json`

### Endpoints

#### GET /api/orders/customer/{customerId}

* **Description:** Retrieve all orders for a specific customer.
* **Tags:** Orders
* **Summary:** Retrieve all orders for a specific customer.
* **Parameters:**
    * `customerId` (path, string, required): Identifier of the customer to retrieve orders for.
* **Responses:**
    * `200`:
        * **Description:** List of customer orders retrieved successfully.
        * **Schema:** `array` of [`Order`](#order)
    * `500`:
        * **Description:** Internal server error (e.g., simulated database failure).
        * **Schema:** [`ErrorResponse`](#errorresponse)
    * `404`:
        * **Description:** Customer not found (if you choose to implement this level of simulation).
        * **Schema:** [`ErrorResponse`](#errorresponse)

#### POST /api/orders

* **Description:** Place a new order.
* **Tags:** Orders
* **Summary:** Place a new order.
* **Parameters:**
    * `OrderRequest` (body, [`OrderRequest`](#orderrequest), required): Order details to create.
* **Responses:**
    * `200`:
        * **Description:** Sample Description
        * **Schema:** [`OrderCreationResponse`](#ordercreationresponse)
    * `400`:
        * **Description:** Invalid request payload.
        * **Schema:** [`ErrorResponse`](#errorresponse)
    * `500`:
        * **Description:** Internal server error (e.g., simulated backend failure).
        * **Schema:** [`ErrorResponse`](#errorresponse)

#### GET /api/orders/{orderId}

* **Description:** Retrieve details of a specific order.
* **Tags:** Orders
* **Summary:** Retrieve details of a specific order.
* **Parameters:**
    * `orderId` (path, string, required): Identifier of the order to retrieve.
* **Responses:**
    * `500`:
        * **Description:** Internal server error (e.g., simulated database failure).
        * **Schema:** [`ErrorResponse`](#errorresponse)
    * `404`:
        * **Description:** Order not found.
        * **Schema:** [`ErrorResponse`](#errorresponse)
    * `200`:
        * **Description:** Order details retrieved successfully.
        * **Schema:** [`Order`](#order)

#### PUT /api/orders/{orderId}

* **Description:** Update the status of an existing order.
* **Tags:** Orders
* **Summary:** Update the status of an existing order.
* **Parameters:**
    * `orderId` (path, string, required): Identifier of the order to retrieve.
    * `OrderStatusUpdateRequest` (body, [`OrderStatusUpdateRequest`](#orderstatusupdaterequest), required): New status for the order.
* **Responses:**
    * `500`:
        * **Description:** Internal server error (e.g., simulated backend failure).
        * **Schema:** [`ErrorResponse`](#errorresponse)
    * `200`:
        * **Description:** Order status updated successfully.
    * `404`:
        * **Description:** Order not found.
        * **Schema:** [`ErrorResponse`](#errorresponse)
    * `400`:
        * **Description:** Invalid request payload.
        * **Schema:** [`ErrorResponse`](#errorresponse)
### Definitions

#### Product

```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "description": "Name of the product."
    },
    "quantity": {
      "type": "integer",
      "description": "Quantity of the product ordered.",
      "minimum": 1
    }
  }
}

{
  "type": "object",
  "required": [
    "customerId",
    "products"
  ],
  "properties": {
    "customerId": {
      "type": "string",
      "description": "Identifier of the customer placing the order."
    },
    "products": {
      "type": "array",
      "description": "List of products in the order.",
      "items": {
        "$ref": "#/definitions/Product"
      }
    }
  }
}
```
---

#### Order

```json
{
  "type": "object",
  "properties": {
    "orderId": {
      "type": "string",
      "description": "Unique identifier of the order.",
      "readOnly": true
    },
    "customerId": {
      "type": "string",
      "description": "Identifier of the customer."
    },
    "orderDate": {
      "type": "string",
      "format": "date-time",
      "description": "Date and time the order was placed.",
      "readOnly": true
    },
    "products": {
      "type": "array",
      "description": "List of products in the order.",
      "items": {
        "$ref": "#/definitions/Product"
      }
    },
    "status": {
      "type": "string",
      "description": "Current status of the order.",
      "enum": [
        "Pending",
        "Processing",
        "Shipped",
        "Cancelled"
      ]
    }
  }
}
```

#### Order Update

```JSON
{
  "type": "object",
  "required": [
    "status"
  ],
  "properties": {
    "status": {
      "type": "string",
      "description": "New status for the order.",
      "enum": [
        "Processing",
        "Shipped",
        "Cancelled"
      ]
    }
  }
}
```

#### Error Response
```JSON
{
  "type": "object",
  "properties": {
    "code": {
      "type": "integer",
      "description": "Error code."
    },
    "message": {
      "type": "string",
      "description": "Error message."
    }
  }
}
```

#### Order Response

```JSON
{
  "type": "object",
  "properties": {
    "orderId": {
      "type": "string",
      "description": "The newly created order ID."
    }
  }
}
```

