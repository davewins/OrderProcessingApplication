# Order Processing Application

This repository contains a simulated **Order Processing Application** built using **TIBCO BusinessWorks** for deployment into the **TIBCO Platform**. The deployment of this application is primarily for the TIBCO Platform and it's designed to show customers what a real-world application might look like within that environment. It's intended to showcase various aspects of the platform, including **Observability**, **Tracing**, **Autoscaling**, **Logging**, and **Application Management**. The application exposes a suite of RESTful services to manage customer orders, simulating a real-world order fulfillment process.

---

## Table of Contents

* [Overview](#overview)
* [Features](#features)
* [REST API Endpoints](#rest-api-endpoints)
* [Key BusinessWorks Processes](#key-businessworks-processes)
* [Simulations](#simulations)
* [TIBCO EMS Integration](#tibco-ems-integration)
* [Setup and Deployment](#setup-and-deployment)
* [Configuration](#configuration)
* [Logging and Monitoring](#logging-and-monitoring)
* [Developer Hub Integration](#developer-hub)
* [Contributing](#contributing)


---

## Overview

The Order Processing Application is designed to demonstrate a comprehensive order management flow within the TIBCO BusinessWorks environment. It provides a set of **REST APIs** for order creation, retrieval, and status updates, while internally simulating database interactions, inventory checks, and audit logging using **TIBCO Enterprise Message Service (EMS)**.

### Important Note on Simulation

Since this application is a simulation for demonstration purposes, the specific values used for `orderId`, `customerId`, and other identifiers in the request and response examples are purely illustrative. The application is designed to process any valid request format, and the exact content of these IDs does not impact the simulated business logic. Feel free to use any fictitious IDs for testing and exploration.

---

## Features

* **RESTful Order Management:** Exposes REST services for creating, retrieving, updating, and querying orders.
* **Simulated Database Operations:** Database interactions (inserts, updates, queries) are simulated using randomized sleep timers.
* **Simulated Audit Logging:** Comprehensive logging of application activities to a simulated audit database via JMS.
* **Inventory Simulation:** Includes a simulated inventory check during the order creation process.
* **TIBCO EMS Integration:** Utilizes JMS Queues for inter-process communication and simulating real-world messaging scenarios.
* **Modular Design:** Organized into multiple BusinessWorks processes for clarity and maintainability.

---

## REST API Endpoints

The application exposes the following RESTful endpoints:

### Place New Order

* `POST /api/orders`
* **Description:** Creates a new order in the system.
* **Request Body Example:**

    ```json
    {
      "customerId": "CUST001",
      "products": [
        {
          "name": "PROD123",
          "quantity": 2
        },
        {
          "name": "PROD456",
          "quantity": 1
        }
      ]
    }
    ```
* **Response Body Example:**

    ```json
    {
      "orderId": "ORD-XYZ-789"
    }
    ```

### Get Order Information

* `GET /api/orders/{orderId}`
* **Description:** Retrieves detailed information for a specific order.
* **Path Parameters:**
    * `orderId`: The unique identifier of the order.
* **Response Body Example:**

    ```json
    {
      "orderId": "ORD-XYZ-789",
      "customerId": "CUST001",
      "orderDate": "2024-05-22T10:30:00Z",
      "products": [
        {
          "name": "PROD123",
          "quantity": 2
        },
        {
          "name": "PROD456",
          "quantity": 1
        }
      ],
      "status": "Shipped"
    }
    ```

### Update Order Status

* `PUT /api/orders/{orderId}`
* **Description:** Updates the status of an existing order.
* **Path Parameters:**
    * `orderId`: The unique identifier of the order.
* **Request Body Example:**

    ```json
    {
      "status": "Shipped"
    }
    ```
* **Response:**
    * `200 OK` (No response body for a successful update)

### Get All Customer Orders

* `GET /api/orders/customer/{customerId}`
* **Description:** Retrieves all orders placed by a specific customer.
* **Path Parameters:**
    * `customerId`: The unique identifier of the customer.
* **Response Body Example:**

    ```json
    [
      {
        "orderId": "ORD-ABC-123",
        "customerId": "CUST001",
        "orderDate": "2024-05-20T09:00:00Z",
        "products": [
          {
            "name": "PROD789",
            "quantity": 1
          }
        ],
        "status": "Completed"
      },
      {
        "orderId": "ORD-XYZ-789",
        "customerId": "CUST001",
        "orderDate": "2024-05-22T10:30:00Z",
        "products": [
          {
            "name": "PROD123",
            "quantity": 2
          },
          {
            "name": "PROD456",
            "quantity": 1
          }
        ],
        "status": "Shipped"
      }
    ]
    ```

---

## Key BusinessWorks Processes

The application is structured around several key BusinessWorks processes:

* `OrdersRESTInterface`: The main interface process that exposes the REST services. It orchestrates calls to various sub-processes based on the incoming request.
* `Activator`: A crucial process that initializes and sets up shared variables used throughout the application. This typically runs on application startup.
* `Audit Log`: This process is responsible for receiving JMS messages containing audit logs and simulating their storage into a database.
* `CreateOrderProcess`: Handles the logic for creating a new order. This includes logging the request, simulating database insertion, performing an inventory check, and sending order details over JMS.
* `DatabaseUpdate`: A sub-process simulating the update of database records with randomized sleep durations.
* `InventoryCheck`: A sub-process that simulates checking the availability of products in inventory.
* `Logger`: A utility sub-process used for generic logging of events and messages.
* `QueryOrder`: A sub-process that simulates querying order information from a database.
* `StockCheck`: A sub-process that simulates checking the stock levels for products.
* `UpdateOrderStatus`: A sub-process that simulates updating the status of an order in the database.

---

## Simulations

To provide a self-contained and runnable demonstration, the application incorporates several simulations:

* **Simulated Database Updates:** All database interactions (inserts, updates, queries) are implemented using **sleep timers with randomized durations**. This simulates the latency and variability of real database operations without requiring an actual database connection.
* **Simulated Audit Database:** Logging to an "audit database" is also simulated using a randomized sleep activity, demonstrating the flow of audit data via TIBCO EMS.

---

## TIBCO EMS Integration

**TIBCO Enterprise Message Service (EMS)** plays a vital role in the application's internal communication and simulation of real-world scenarios. The following queues must be defined and accessible within your TIBCO EMS environment **before deploying the application**:

* `Orders`
* `LogMessage`
* `StockCheck`

---

## Setup and Deployment

### Prerequisites

* **TIBCO BusinessWorks Container Edition capability**: Deployed in your TIBCO Platform Dataplane.
* **TIBCO EMS**: Deployed and configured with the necessary queues (`Orders`, `LogMessage`, `StockCheck`, `AuditQueue`) within your TIBCO Platform Dataplane.
* **TIBCO BusinessStudio**: Downloaded and installed from the TIBCO Platform.

### Building and Deploying the Application

You have two primary options for building and deploying this application:

1.  **Build `.ear` file using BusinessStudio:**
    * Open the project in **TIBCO BusinessWorks Studio**.
    * Perform a "Clean and Build" of the project to ensure all dependencies are resolved.
    * Export the application as a `.ear` file. You will then manually deploy this `.ear` in the next step.

2.  **"Push to Platform" for Direct Deployment:**
    * Within **TIBCO BusinessWorks Studio**, utilize the "Push to Platform" option. This will directly build and deploy the application to your configured TIBCO Platform environment. If you choose this option, you can skip the "Deployment to TIBCO Platform" section below.

### Deployment to TIBCO Platform

*This section is only necessary if you chose Option 1 (building the `.ear` file) in the "Building and Deploying the Application" step above.*

1.  Log in to your **TIBCO Platform** and navigate to the appropriate Data Plane where you wish to deploy the application.
2.  Use the Platform's deployment tools to create a new BusinessWorks application.
3.  Upload the `.ear` file built in the previous step.

From this point, the configuration of the application properties is the same regardless of whether you deployed via `.ear` upload or "Push to Platform".

---

## Configuration

There are a number of application variables that need to be configured after deployment. Primarily, you will need to set the **EMS connection details**, which will be dependent on your EMS configuration within the TIBCO Platform.

* **TIBCO EMS Connection Details:**
    * `EMSURL`: e.g. `http://ems-emsactive.dw.svc:9011,tcp://ems-ems.dw.svc:9011`
    * `EMSUserName`: `admin`
    * `EMSPassword`: `---`
* **Logging Level:**
    * `LogLevel`: Can be set to `INFO` for standard logging.


---

## Developer-hub

In order to import the catalog-info.yaml file into your Developer Hub instance, you do need to make sure that the appropriate rules are implemented in your Developer Hub Configuration to allow the full import to take place.

* In your dataplane, go to your Developer Hub Capability, and click on "Update Configuration"
* In the configuration file, add the following:
```yaml
catalog:
  import:
    entityFilename: catalog-info.yaml
  rules:
    - allow:
        - Component
        - API
        - Location
        - System
        - Resource
        - Group
        - User
        - Domain
```

Once the configuration has been updated, you will then be able to use the "Import" flow to import the catalog-info.yaml file from this repository.

---

## Contributing

Contributions are welcome! If you find a bug or have an improvement, please open an issue or submit a pull request.
