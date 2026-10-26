---
name: bw6-prompt-library
description: Browse-only catalog of the BW6 sample-application prompt skills. Use when the user wants to see what BW6 prompts/examples are available without running one — e.g. "list the BW6 prompts", "what retail samples do we have", "show the prompt library", "which BW6 skill fits my use case", "prompt catalog", "browse prompts", "prompt library index", "what sample can I start from". Does NOT scaffold or execute any project — only lists categories, names, and one-liners so the user can pick, then instructs how to invoke the chosen skill.
---

# BW6 Prompt Library — Catalog (browse only)

Read-only index of the seven BW6 sample-application prompts installed as skills. Do **not** create files, call `bwdesign`, or invoke any `mcp__bw__*` tool from this skill — it exists purely to help the user pick which prompt they want, then hand off.

## How to use this skill

When triggered, render the catalog table below verbatim and then say:

> To run one, name the sample (e.g. *"scaffold the retail JDBC inventory example"*) or invoke the skill directly (e.g. `/jdbcRetailInventoryDb`). Each build is checked against the `bw6-rules` skill and driven through the `bwdesign` skill.

Do not automatically load the chosen skill — wait for the user's confirmation, since they may just be browsing.

## Catalog

| # | Category | Skill | What it builds | Main tech |
|---|---|---|---|---|
| 1 | **Basic** | [`basicRetailOrderLogger`](basicRetailOrderLogger/SKILL.md) | `RetailOrderLogger` — Timer → Log → Mapper → CallProcess sample with module properties, a property group, and a `ValidateOrder` subprocess. | Timer, Log, Mapper, Call Process |
| 2 | **JDBC** | [`jdbcRetailInventoryDb`](jdbcRetailInventoryDb/SKILL.md) | `retail.bw.jdbc.InventoryUpdate` — scheduled JDBC Query + Update against Postgres `retaildb` with a shared `RetailDBConnection` and file logging. | JDBC Query, JDBC Update, Write File |
| 3 | **JMS** | [`jmsRetailOrderQueue`](jmsRetailOrderQueue/SKILL.md) | `RetailOrderQueue` — JMS Receive on `retail.orders.queue`, log + write to file, Reply to JMS. Sets up JNDI + JMS Connection shared resources against TIBCO EMS. | JNDI, JMS Connection, JMS Receive, Reply to JMS |
| 4 | **REST** | [`restRetailProductService`](restRetailProductService/SKILL.md) | `RetailProductService` — Swagger-enabled REST service with GET `/products/{productId}` and POST `/products` backed by `Product.xsd`. | REST Service, HTTP Request/Response, Swagger |
| 5 | **File** | [`fileRetailSalesFile`](fileRetailSalesFile/SKILL.md) | `retail.bw.sample.palette.file.RetailDailySales` — Timer → Mapper (against `RetailSalesSchema.xsd`) → Write File (append) → Log. | File Poller / Write File, Mapper |
| 6 | **SubProcess** | [`subprocessRetailReturns`](subprocessRetailReturns/SKILL.md) | `retail.bw.sample.palette.subprocess.RetailReturnProcess` — main process delegates return validation to `ValidateReturn.bwp` via CallProcess. | Timer, Call Process, Mapper |
| 7 | **SOAP** | [`soapRetailLoyaltyService`](soapRetailLoyaltyService/SKILL.md) | `retailLoyaltyService` — SOAP service generated from `LoyaltyService.wsdl` (`GetLoyaltyPoints`) with auto-generated SOAP Receive/Reply plus Mapper + Log. | WSDL, SOAP Receive, SOAP Reply |

## Companion skills

- **`bwdesign`** — drives the actual BW6 build (CLI or `mcp__bw__*` when Business Studio is open).
- **`bw6-rules`** — 62 sonar-bw quality rules applied advisory-style during and after each build.
