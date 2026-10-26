# Order Processing Application

This application is a core component of the order management system, responsible for orchestrating the processing of new customer orders. It leverages TIBCO BusinessWorks to integrate with various internal and external systems.

## Key Features
* Receives new order requests via a TIBCO EMS queue.
* Performs data validation and enrichment.
* Interacts with a transactional database for order persistence.
* Publishes order status updates to another EMS queue.

## Architecture Overview
[Link to Architecture diagram](architecture.md)