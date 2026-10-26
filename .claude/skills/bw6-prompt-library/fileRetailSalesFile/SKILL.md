---
name: fileRetailSalesFile
description: Build the "retail.bw.sample.palette.file.RetailDailySales" BW6 application — a Timer-driven process that maps sales data via `RetailSalesSchema.xsd`, appends a summary line to an output file, and logs the result. Use when the user asks to create/scaffold the retail daily sales file example, needs a BW6 File palette sample with Write File + Mapper, or references any of: "retail daily sales", "RetailDailySales", "Write File", "sales_summary.txt", "SalesFileProcessor.bwp", "file palette sample". Drives the build via `bwdesign` / `mcp__bw__*` tools and applies `bw6-rules` checks.
---

# RetailDailySales — File Palette Sample (BW6)

Introduces the **File palette** (`Write File`) alongside a **Timer starter**, **Mapper** against an XSD, and **Log**, plus **process properties** that shadow module properties.

Category: **File** • Main tech: `File Poller, Parse Data, Mapper, Write File` (this sample uses Timer instead of File Poller).

## How to run this skill

1. Confirm the target BW workspace with the user. Prefer `mcp__bw__*` if Business Studio is open, else `bwdesign` per the `bwdesign` skill.
2. Execute the spec below step by step. Announce each major step before invoking the tool.
3. Run `bw6-rules` in advisory mode. Rules that could fire here:
   - `JMSHardCoded`/`JDBCHardCoded` — n/a for this sample.
   - `ProcessNoDescription`, `ProcessNamingConvention`, `TransitionLabels` — populate descriptions and labels where you add them.
   - `BwSharedResourceUsingModuleProperty` — this sample already binds `Write File` `FileName` to a module property, keep it that way.
4. Validate after implementation and report status.

## Project Specification

### Project Hierarchy

| Component Type | Name |
| :---- | :---- |
| **Application Module** | `retail.bw.sample.palette.file.RetailDailySales` |
| **Application Project** | `retail.bw.sample.palette.file.RetailDailySales.application` |

### Module Properties

| Property Name | Data Type | Value |
| :---- | :---- | :---- |
| OUTPUT_FILE | String | `C:/Retail/Output/sales_summary.txt` |
| DEFAULT_STORE_NAME | String | RetailMart |
| DEFAULT_TOTAL_SALES | String | 15000.00 |
| DEFAULT_SALES_DATE | String | 2025-01-01 |

### Process Properties for `SalesFileProcessor.bwp`

| Property Name | Data Type | Value (source module prop) |
| :---- | :---- | :---- |
| storeName | String | DEFAULT_STORE_NAME |
| totalSales | String | DEFAULT_TOTAL_SALES |
| salesDate | String | DEFAULT_SALES_DATE |
| OUTPUT_FILE | String | OUTPUT_FILE |

### Schema `RetailSalesSchema.xsd`

Elements: `storeName` (xs:string), `totalSales` (xs:decimal), `salesDate` (xs:date).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           targetNamespace="http://www.tibco.com/retail/sales"
           xmlns:tns="http://www.tibco.com/retail/sales"
           elementFormDefault="qualified">
    <xs:element name="RetailSales" type="tns:RetailSalesType"/>
    <xs:complexType name="RetailSalesType">
        <xs:sequence>
            <xs:element name="storeName"  type="xs:string"/>
            <xs:element name="totalSales" type="xs:decimal"/>
            <xs:element name="salesDate"  type="xs:date"/>
        </xs:sequence>
    </xs:complexType>
</xs:schema>
```

### Process `SalesFileProcessor.bwp`

Activities: `Timer` → `Mapper` → `Write File` → `Log`. Link in sequence.

- **Timer (Starter)** — `Interval` = `60` seconds.
- **Mapper** — schema `RetailSalesSchema.xsd`:
  - `storeName` ← `$storeName`
  - `totalSales` ← `xsd:decimal($totalSales)`
  - `salesDate` ← `xsd:date($salesDate)`
- **Write File**:
  - `FileName` = module property `OUTPUT_FILE`
  - `Append` = `true`
  - `Create Non Existing Directories` = `true`
  - `textContent` = `concat("Store: ", $Mapper/tns:storeName, " | Date: ", $Mapper/tns:salesDate, " | Total Sales: ", string($Mapper/tns:totalSales))`
- **Log** — `message` = `concat("Retail sales record processed for store: ", $Mapper/tns:storeName, " on ", $Mapper/tns:salesDate)`
