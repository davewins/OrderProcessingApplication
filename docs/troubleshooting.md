# Troubleshooting Guide

This guide provides common troubleshooting steps and known issues for the Order Processing Application.

## 1. General Troubleshooting Steps

Before escalating, please check the following:

1.  **Application Logs:** Review the logs for the `OrderProcessingApplication` BWCE instance. Look for `ERROR` or `WARN` level messages.
2.  **TIBCO EMS Queues:**
    * Verify `queue.order.new` is receiving messages.
    * Verify `queue.order.status` is publishing messages.
    * Check for pending messages or consumers on relevant queues using EMS Admin or a monitoring tool.
3.  **External Service Connectivity:**
    * **Payment Gateway:** Ensure network connectivity to `api.paymentgateway.com`. Check firewall rules if applicable.
    * **CRM Service:** Ensure connectivity to `crm-service.internal`.
4.  **Database Connectivity:**
    * Verify the application can connect to `order-database-primary`.
    * Check database server status and connection limits.
5.  **Kubernetes Pod Status:** If deployed on Kubernetes, check the pod status:
    ```bash
    kubectl get pods -n <your-namespace> -l app=order-processing-app
    kubectl logs -f <order-processing-app-pod-name> -n <your-namespace>
    ```

## 2. Common Issues and Solutions

### 2.1. Orders Stuck in `queue.order.new`

* **Symptom:** Messages are accumulating on `queue.order.new`, but no new orders are being processed.
* **Possible Causes:**
    * `OrderReceiver` BW process not running or consumer is down.
    * EMS connection issues from the BW application.
    * BW application overwhelmed / resource constraints.
* **Solutions:**
    1.  **Check BWCE Instance:** Verify the `OrderProcessingApplication` BWCE instance is running and healthy.
    2.  **Restart BWCE App:** If the app is unhealthy, try restarting the BWCE application pod.
    3.  **EMS Connectivity:** Check BW logs for EMS connection errors. Verify EMS broker is up and accessible.
    4.  **Resource Utilization:** Monitor CPU/memory usage of the BWCE pod. Scale up resources if needed.

### 2.2. Payment Authorization Failures

* **Symptom:** Orders are failing during the payment authorization step.
* **Possible Causes:**
    * Invalid credentials for Payment Gateway API.
    * Network connectivity issues to Payment Gateway.
    * Payment Gateway API rate limits exceeded.
    * Invalid payment method data in the incoming order.
* **Solutions:**
    1.  **Check BW Logs:** Look for specific error messages from the `PaymentService` activity, including HTTP status codes from the Payment Gateway.
    2.  **Verify API Key/Credentials:** Confirm the configured payment gateway API key/credentials are correct and active.
    3.  **Network Check:** Ping/telnet the payment gateway endpoint from the BWCE host/pod.
    4.  **Payment Gateway Dashboard:** Check the payment gateway's own dashboard for transaction logs or service outages.

### 2.3. Database Insert/Update Errors

* **Symptom:** Orders fail to persist or update their status in the database.
* **Possible Causes:**
    * Database connection issues (credentials, network).
    * Database server is down or overloaded.
    * Schema mismatch (BW application trying to insert data that doesn't fit the table structure).
    * Deadlocks or contention.
* **Solutions:**
    1.  **Check BW Logs:** Look for JDBC errors, SQL exceptions, or connection pool issues.
    2.  **Database Status:** Verify the `order-database-primary` database is running and accessible.
    3.  **Database Logs:** Review database server logs for errors (e.g., connection limits, disk space, query performance).
    4.  **Schema Validation:** If a recent deployment occurred, confirm the database schema matches the application's expectations.

### 2.4. Orders Not Appearing in Fulfillment/Notifications

* **Symptom:** Orders are processed by the BW app, but downstream services (Fulfillment, Notification) are not receiving updates.
* **Possible Causes:**
    * `Order Status Queue` (TIBCO EMS) issues.
    * Downstream services not consuming from `queue.order.status`.
    * Incorrect message format being published.
* **Solutions:**
    1.  **Check `queue.order.status`:** Verify messages are being published to this queue by the BW app.
    2.  **Verify Downstream Consumers:** Check the status and logs of the Fulfillment and Notification services to ensure they are connected and consuming from the queue.
    3.  **Message Inspection:** Use EMS Admin or a tool to inspect messages on `queue.order.status` to ensure the format is correct and expected.

## 3. Known Issues

* **Simulated Database Calls:** In non-production environments, database calls are simulated with randomized `sleep` activities. This can lead to unpredictable delays in processing times. This is intentional for development and testing.
* **High Latency with CRM Lookups:** Occasional high latency observed when calling the CRM Customer Lookup API during peak hours. This is an upstream issue being investigated by the CRM team. The BW app has configured timeouts to handle this gracefully.

## 4. Escalation Matrix

If you are unable to resolve the issue using the steps above, please escalate to:

1.  **Level 1 Support:** Your immediate team or designated L1 support channel.
2.  **Level 2 Support:** The `Order Processing Team` (refer to the [team page in Backstage](https://hub.vewins.co.uk/tibco/hub/groups/default/order-processing-team) for contact details).
3.  **Critical Incident:** For P1/Sev1 issues impacting production, follow the standard incident management process.

---

Remember to commit these files and push them to your GitHub repository, then refresh/re-ingest your catalog-info in Backstage!