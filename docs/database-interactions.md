# Database Interactions

This application performs several operations against the `OrderProcessingDB` (a fictitious database for demonstration purposes).

## Tables Accessed
* `orders`: Stores primary order details.
* `order_items`: Details for each item in an order.
* `customer_info`: Basic customer reference data (read-only).

## Operations
* **Insert Order:** Upon receiving a new order, a new record is inserted into the `orders` and `order_items` tables.
* **Update Order Status:** The `status` field in the `orders` table is updated as the order progresses.
* **Query Customer Data:** Random sleep activities simulate database lookups for customer details during validation.

## Simulated Database Calls
In this development environment, database calls are simulated using randomized sleep activities to represent latency and processing time. In a production environment, these would be actual JDBC or similar database connectors.