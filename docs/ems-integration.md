# TIBCO EMS Integration

The Order Processing Application interacts with TIBCO EMS for both input and output messaging.

## Input Queue: `queue.order.new`
* **Purpose:** Receives new order requests from upstream systems.
* **Message Format:** JSON payload conforming to the `OrderRequestSchema`.
* **Durability:** Persistent messages, guaranteed delivery.

## Output Queue: `queue.order.status`
* **Purpose:** Publishes real-time updates on order status (e.g., `RECEIVED`, `PROCESSING`, `COMPLETED`, `FAILED`).
* **Message Format:** JSON payload conforming to the `OrderStatusSchema`.
* **Listeners:** Used by fulfillment and notification services.