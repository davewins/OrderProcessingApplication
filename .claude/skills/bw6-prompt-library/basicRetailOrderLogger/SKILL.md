---
name: basicRetailOrderLogger
description: Build the "RetailOrderLogger" BW6 sample application — a basic Timer-driven process that logs an order, maps it against a RetailOrderSchema.xsd, and calls a ValidateOrder subprocess. Use this when the user asks to create/scaffold the retail order logger, needs a starter BW6 "hello world" with module properties + property groups + subprocess call, or references any of: "retail order logger", "RetailOrderLogger", "basic BW6 sample", "Timer Log Mapper CallProcess", "OrderProcess.bwp", "ValidateOrder.bwp". Drives the build via `bwdesign` / `mcp__bw__*` tools and applies `bw6-rules` checks.
---

# RetailOrderLogger — Basic BW6 Sample

Basic BW6 pattern that introduces **module properties (incl. a property group)**, **XSD schema**, **Timer starter**, **Log**, **Mapper**, and **subprocess call**.

Category: **Basic** • Main tech: `Timer, Log, Mapper, Call Process`.

## How to run this skill

1. Confirm with the user which BW workspace to build in. If Business Studio is running, prefer `mcp__bw__*` tools; otherwise shell out to `bwdesign` per the `bwdesign` skill.
2. Execute the spec below step by step. Announce each major step before you run the tool.
3. Cross-check every change against the `bw6-rules` skill and surface findings advisory-style. Notable rules that will fire on this template:
   - `ProcessNamingConvention`, `ProcessNoDescription` — populate descriptions.
   - `DefaultTargetNamespace` — the given XSD sets a real namespace, keep it.
   - `TransitionLabels` — this template uses unconditional links so labels are not required.
   - `AtLeastOneStarter` — the Timer starter satisfies this.
4. When finished, run `system:validate` (CLI) or `mcp__bw__getCompilationErrors` and report status.

## Project Specification

### Project Overview

| Field | Value |
| :---- | :---- |
| **Application Module Name** | `RetailOrderLogger` |
| **Application Name** | `RetailOrderLogger.application` |

### A. Module Properties Configuration

Create and configure the following module properties:

| Property Name | Data Type | Value |
| :---- | :---- | :---- |
| orderStoreId | Long | 10001 |
| defaultOrderId | Integer | 501 |
| enableOrderValidation | Boolean | true |
| orderApiPassword | Password | Retail@123 |

Create a new group **`retailGroup`** with:

| Property Name | Data Type | Value |
| :---- | :---- | :---- |
| storeName | String | RetailMart |

### B. Schemas and Resources

Add schema **`RetailOrderSchema.xsd`**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           targetNamespace="http://www.tibco.com/retail/order"
           xmlns:tns="http://www.tibco.com/retail/order"
           elementFormDefault="qualified">

    <xs:element name="RetailOrder" type="tns:RetailOrderType"/>

    <xs:complexType name="RetailOrderType">
        <xs:sequence>
            <xs:element name="orderName"    type="xs:string"/>
            <xs:element name="orderId"      type="xs:integer"/>
            <xs:element name="storeId"      type="xs:long"/>
            <xs:element name="orderDate"    type="xs:dateTime"/>
            <xs:element name="orderStatus"  type="xs:string"/>
            <xs:element name="customerInfo" type="tns:CustomerInfoType"/>
            <xs:element name="orderItems"   type="tns:OrderItemsType"/>
            <xs:element name="orderTotal"   type="xs:decimal"/>
        </xs:sequence>
    </xs:complexType>

    <xs:complexType name="CustomerInfoType">
        <xs:sequence>
            <xs:element name="customerId"    type="xs:integer"/>
            <xs:element name="customerName"  type="xs:string"/>
            <xs:element name="customerEmail" type="xs:string"/>
            <xs:element name="customerPhone" type="xs:string"/>
        </xs:sequence>
    </xs:complexType>

    <xs:complexType name="OrderItemsType">
        <xs:sequence>
            <xs:element name="item" type="tns:OrderItemType" maxOccurs="unbounded"/>
        </xs:sequence>
    </xs:complexType>

    <xs:complexType name="OrderItemType">
        <xs:sequence>
            <xs:element name="itemId"       type="xs:integer"/>
            <xs:element name="itemName"     type="xs:string"/>
            <xs:element name="quantity"     type="xs:integer"/>
            <xs:element name="unitPrice"    type="xs:decimal"/>
            <xs:element name="totalPrice"   type="xs:decimal"/>
        </xs:sequence>
    </xs:complexType>

</xs:schema>
```

### C. Process Architecture

| Package Name | Process Name |
| :---- | :---- |
| `retailorderlogger` | `OrderProcess.bwp` |
| `retailorderlogger` | `ValidateOrder.bwp` |

### D. Process Implementation

**1. `OrderProcess.bwp`** — Activities: `Timer` → `Log` → `Mapper` → `Log1` → `CallProcess` → `Log2`. Link in sequence.

- **Log** → `Input > ActivityInput > message`: `$Timer/Time`
- **Mapper** → schema `RetailOrderSchema.xsd`, then:
  - `orderName` ← `$_processContext/ApplicationName`
  - `orderId`   ← `xsd:integer(bw:getModuleProperty("defaultOrderId"))`
- **Log1** → `message`: `$Mapper/tns:orderName`
- **CallProcess** → `Process Name`: `ValidateOrder.bwp`
- **Log2** → `message`: `bw:getModuleProperty("orderApiPassword")`

**2. `ValidateOrder.bwp`** — Activities: `Start` → `Log` → `Log1` → `End`. Link in sequence.

- **Log** → `message`: `bw:getModuleProperty("BW.PROCESS.NAME")`
- **Log1** → `message`: `bw:getModuleProperty("/retailGroup/storeName")`
