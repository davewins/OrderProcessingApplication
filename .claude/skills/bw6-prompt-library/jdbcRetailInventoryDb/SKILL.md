---
name: jdbcRetailInventoryDb
description: Build the "retail.bw.jdbc.InventoryUpdate" BW6 application — a scheduled Timer + JDBC Query + JDBC Update process against a Postgres retail DB, writing the update count to a log file. Use when the user asks to create/scaffold the retail JDBC inventory example, needs a BW6 JDBC palette sample with a shared `JDBCConnectionResource`, or references any of: "retail inventory", "InventoryUpdate", "JDBC Query", "JDBC Update", "RetailDBConnection", "PRODUCT_TABLE", "retaildb", "postgresql". Drives the build via `bwdesign` / `mcp__bw__*` tools and applies `bw6-rules` checks.
---

# retail.bw.jdbc.InventoryUpdate — JDBC Palette Sample (BW6)

Introduces a **JDBC Connection shared resource**, **JDBC Query** with parameters, **JDBC Update**, and file logging.

Category: **JDBC** • Main tech: `JDBC Query, JDBC Update, Write File`.

## How to run this skill

1. Confirm the target BW workspace with the user. Prefer `mcp__bw__*` if Business Studio is open, else `bwdesign` per the `bwdesign` skill.
2. Execute the spec below step by step. Announce each major step.
3. Cross-check against `bw6-rules`. Rules that WILL fire without extra care:
   - `JDBCHardCoded` — this sample does not set `Timeout` / `MaxRows`; either bind them to a Module Property or explicitly acknowledge the omission.
   - `JDBCWildcards` — the queries here list explicit columns, good.
   - `ThreadpoolUsageInJDBCActivities` — the JDBC activities need a `ThreadPool Resource`; add one under Advanced tab.
   - `BwSharedResourceUsingModuleProperty` — the shared `RetailDBConnection` binds all attrs to module properties, good.
4. Run `system:validate` / `mcp__bw__getCompilationErrors` and report status.

## Project Specification

### Project Hierarchy

| Component Type | Name |
| :---- | :---- |
| **Application Module** | `retail.bw.jdbc.InventoryUpdate` |
| **Application Project** | `retail.bw.jdbc.InventoryUpdate.application` |

### Module Properties

| Property Name | Data Type | Value |
| :---- | :---- | :---- |
| USERNAME | String | retail_user |
| jdbc_PASSWORD | Password | Retail@123 |
| JDBC_URL | String | `jdbc:postgresql://localhost:5432/retaildb` |
| JDBCConnectionResource | jdbc | `retailapp.RetailDBConnection` |
| JDBC_DRIVER | String | `org.postgresql.Driver` |
| PRODUCT_ID | Integer | 501 |
| OUTPUT_FILE | String | `c:/tmp/RetailInventory/inventory_update.log` |

### Process Properties for `Process.bwp`

| Property Name | Data Type | Value (source module prop) |
| :---- | :---- | :---- |
| OUTPUT_FILE | String | OUTPUT_FILE |
| productId | Integer | PRODUCT_ID |

### Shared Resource `RetailDBConnection`

Fully-qualified name: `retail.bw.jdbc.InventoryUpdate.RetailDBConnection`

- `Username` ← module property `USERNAME`
- `Password` ← module property `jdbc_PASSWORD`
- `Database Driver` ← module property `JDBC_DRIVER`
- `Database URL` ← module property `JDBC_URL`

### Process `Process.bwp`

Activities: `Timer` → `JDBC Query` → `JDBC Update` → `Write File` → `Log`. Link in sequence.

- **Timer (Starter)**.
- **JDBC Query**
  - `JDBC Shared Resource` = `retail.bw.jdbc.InventoryUpdate.RetailDBConnection`
  - **SQL:** `SELECT PRODUCT_ID, PRODUCT_NAME, STOCK_QUANTITY FROM PRODUCT_TABLE WHERE PRODUCT_ID = ?`
  - Parameter: `productId` (INTEGER)
  - Input node: `productId` ← `$productId`
- **JDBC Update**
  - **SQL:** `UPDATE PRODUCT_TABLE SET STOCK_QUANTITY = STOCK_QUANTITY - 1 WHERE PRODUCT_ID = ?`
  - Parameter: `productId` (INTEGER)
  - Input node: `jdbcUpdateActivityInput` ← `$JDBCQuery/Record[1]`
- **Write File**
  - `FileName` = module property `OUTPUT_FILE`
  - `textContent` = `concat("inventory records updated: ", string($JDBCUpdate/noOfUpdates))`
- **Log**
  - `message` = `concat("Retail Inventory Process Complete: Updated stock for ", xsd:string($JDBCQuery/Record[1]/PRODUCT_NAME))`
