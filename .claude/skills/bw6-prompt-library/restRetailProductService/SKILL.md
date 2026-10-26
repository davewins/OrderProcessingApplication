---
name: restRetailProductService
description: Build the "RetailProductService" BW6 application — a REST service with Swagger enabled that exposes GET `/products/{productId}` and POST `/products` operations backed by `Product.xsd`. Use when the user asks to create/scaffold the retail product REST service, needs a BW6 REST palette sample with `Receive HTTP Request` + `Send HTTP Response`, or references any of: "retail product service", "RetailProductService", "REST Service", "Product.xsd", "GET Product", "POST Product", "ProductAPIProcess.bwp", "Swagger". Drives the build via `bwdesign` / `mcp__bw__*` tools and applies `bw6-rules` checks.
---

# RetailProductService — REST Palette Sample (BW6)

Introduces a **REST Service** with **Swagger documentation** enabled, exposing GET and POST operations mapped against a `Product.xsd` schema.

Category: **REST** • Main tech: `REST Service, HTTP Request/Response, Swagger`.

## How to run this skill

1. Confirm the target BW workspace with the user. Prefer `mcp__bw__*` if Business Studio is open, else `bwdesign` per the `bwdesign` skill.
2. Execute the spec below step by step. Announce each major step before invoking a tool.
3. Cross-check against `bw6-rules`. Rules to watch:
   - `SwaggerValidation` — after the Swagger is generated, validate it.
   - `BindingShouldHavePolicyAssociated` / `BindingShouldNotHaveHTTPBasicPolicyAssociated` — the sample does not attach a policy; if the user targets non-dev, prompt them to add a secure policy (not HTTP Basic).
   - `HttpConnectorShouldHaveConfidentiality` — if HTTPS is used, enable confidentiality on the HTTP Connector.
   - `EndpointURIFromHTTPBindingSetUsingProperty` — bind the binding's endpoint URI to a Module Property before deployment.
   - `HttpClientMustBeUsedinHTTPBinding` — for reference bindings only, n/a for this server-side sample.
4. Validate and report status.

## Project Specification

### Project Hierarchy

| Component Type | Name |
| :---- | :---- |
| **Application Module** | `RetailProductService` |
| **Application Project** | `RetailProductService.application` |

### Schema `Product.xsd`

Elements: `productId` (xs:integer), `productName` (xs:string), `productPrice` (xs:decimal), `category` (xs:string).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           targetNamespace="http://www.tibco.com/retail/product"
           xmlns:tns="http://www.tibco.com/retail/product"
           elementFormDefault="qualified">

    <xs:element name="Product" type="tns:ProductType"/>

    <xs:complexType name="ProductType">
        <xs:sequence>
            <xs:element name="productId"    type="xs:integer"/>
            <xs:element name="productName"  type="xs:string"/>
            <xs:element name="productPrice" type="xs:decimal"/>
            <xs:element name="category"     type="xs:string"/>
        </xs:sequence>
    </xs:complexType>

</xs:schema>
```

### REST Service `ProductService`

- `Enable Swagger Documentation` = `true`

#### Operations

| Operation | HTTP Method | Path | Description |
| :---- | :---- | :---- | :---- |
| GET Product | GET | `/products/{productId}` | Returns sample product details |
| POST Product | POST | `/products` | Accepts and processes an incoming product |

**GET Product**
- `Response Schema` = `Product.xsd`
- `productId` ← `$ReceiveHTTPRequest/Parameters/productId`
- `productName` ← `"Sample Product"`
- `productPrice` ← `xsd:decimal("99.99")`
- `category` ← `"General"`

**POST Product**
- `Request Schema` = `Product.xsd`
- `Response Schema` = `Product.xsd`
- `productId` ← `$ReceiveHTTPRequest/Body/tns:Product/tns:productId`
- `productName` ← `$ReceiveHTTPRequest/Body/tns:Product/tns:productName`
- `productPrice` ← `$ReceiveHTTPRequest/Body/tns:Product/tns:productPrice`
- `category` ← `$ReceiveHTTPRequest/Body/tns:Product/tns:category`

### Process `ProductAPIProcess.bwp`

Activities: `Receive HTTP Request` → `Mapper` → `Log` → `Send HTTP Response`. Link in sequence.

- **Receive HTTP Request (Starter)**
  - `REST Service` = `RetailProductService.ProductService`
  - `Operation` = the respective GET or POST operation
- **Mapper** — schema `Product.xsd`. Node bindings mirror the operation:
  - **GET:** `productId` ← `$ReceiveHTTPRequest/Parameters/productId`; `productName` ← `"Sample Product"`; `productPrice` ← `xsd:decimal("99.99")`; `category` ← `"General"`
  - **POST:** `productId` ← `$ReceiveHTTPRequest/Body/tns:Product/tns:productId`; `productName` ← `$ReceiveHTTPRequest/Body/tns:Product/tns:productName`; `productPrice` ← `$ReceiveHTTPRequest/Body/tns:Product/tns:productPrice`; `category` ← `$ReceiveHTTPRequest/Body/tns:Product/tns:category`
- **Log** — `message` = `concat("Retail product request: ", $ReceiveHTTPRequest/Body/tns:Product/tns:productName)`
- **Send HTTP Response**
  - `Response Code` = `200`
  - `Body` = `$Mapper/tns:Product`
