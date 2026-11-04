# Schemas

7 schema files.

| Schema | Elements | Types |
|---|---|---|
| `Schema.xsd` | Element | - |
| `RESTSchema.xsd` | messageBody, httpHeaders, httpResponseHeaders, httpFaultHeaders, dynamicConfigurations, statusLine, client4XXError, server5XXError, jwtClaims | tmessageBody, httpTransportHeaders, httpTransportResponseHeaders, httpTransportFaultHeaders, dynamicHeadersTypeDetails, dynamicHeadersType, dynamicConfigurationsType, statusLineType, client4XXErrorType, server5XXErrorType, jwtClaimElementType, jwtClaimsType |
| `OrdersParameterSchema.xsd` | orders1GetParameters, orders1PutParameters, orders-customerGetParameters | - |
| `OrdersAPI.xsd` | OrderCreationResponse_orderId, OrderRequest_customerId, ErrorResponse, ErrorResponse_code, Product, ErrorResponse_message, OrderStatusUpdateRequest_status, OrderArray, Order, Order_customerId, Product_quantity, OrderCreationResponse (+8 more) | OrderCreationResponse_orderId, OrderRequest_customerId, ErrorResponse, ErrorResponse_code, Product, ErrorResponse_message, OrderStatusUpdateRequest_status, OrderArray, Order, Order_customerId, Product_quantity, OrderCreationResponse (+8 more) |
| `OrderStatus.xsd` | OrderStatus | - |
| `LoggingSchema.xsd` | Element | - |
| `CustomersSchema.xsd` | customers | CustomerType |


