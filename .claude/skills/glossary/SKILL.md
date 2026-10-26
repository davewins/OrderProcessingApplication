---
name: glossary
description: Translate plain-English integration/workflow vocabulary into canonical TIBCO BusinessWorks 6.x terminology, artefact names, and MCP tool names. Use BEFORE acting on any user prompt that describes an integration in generic terms — "app", "workflow", "process", "database call", "queue listener", "config property", "secret", "REST endpoint", "SOAP service", "file writer", "scheduler", "logger", "map/transform", "wire A to B" — so downstream planning uses BW6 nouns (Application Module, BW Process, Module Property, JDBC/JMS/HTTP Shared Resource, palette activities, Transitions) and the correct `mcp__bw__*` tool for each step. Also use when the user mentions rival-platform terms (Mule flow, Boomi process, Camel route, Kafka consumer, Spring Bean, Lambda) — normalise them into BW6 equivalents. Do NOT build anything; this skill only rewrites the ask.
---

# BW6 Glossary — plain-English → BusinessWorks 6.x

Purpose: **translate**, not build. When the user's prompt uses generic or rival-platform vocabulary, rewrite it into the canonical BW6 nouns, artefact names, and MCP tool names below, then hand the normalised ask to the downstream skill (`bw6design`, one of the `pattern-*` skills, or a sample-app skill like `jdbcRetailInventoryDb`).

## How to use this skill

1. Read the user's prompt and mark every generic term (app, workflow, DB call, secret, queue, etc.).
2. Look each term up in the tables below and substitute the BW6 concept + the canonical MCP tool.
3. Emit a **restated prompt** in BW6 language before invoking any builder skill. If a term isn't in the tables, flag it and ask the user rather than guessing.
4. Never call discovery tools (`ListActivity`, `explainProject`, `suggestActivities`, `searchTibcoPdfDocuments`) — this skill is dictionary-only.

Output shape (example):

> **User asked:** "build a scheduled job that reads a row from Postgres, updates it, writes the count to a file, and logs done"
>
> **BW6 restatement:** Build an **Application Module + Application Project**. Add a **BW Process (`.bwp`)** with these palette activities linked in sequence: **Timer** (`bw.generalactivities`) → **JDBC Query** (`bw.jdbc`) → **JDBC Update** (`bw.jdbc`) → **Write File** (`bw.file`) → **Log** (`bw.generalactivities`). DB creds go into **Module Properties** bound to a **JDBC Connection Shared Resource**; process-scope inputs are **Process Properties** bound to those Module Properties. Tools: `createBWApplicationProject`, `createModuleProperty`, `createJDBCConnection`, `bindSharedResourceAttributeToProperty`, `createBWProcess`, `createProcessProperty`, `createActivity`, `createLink`, `configureActivity*`, `getCompilationErrors`.

## 1. Structural nouns

| Plain English / rival-platform term | BW6 concept | Canonical MCP tool |
| :--- | :--- | :--- |
| App, application, service, deployable unit, microservice | **Application Module + Application Project** (the module holds design; the `.application` project is the deployable wrapper) | `createBWApplicationProject` |
| Workflow, process, flow (Mule), route (Camel), pipeline, orchestration | **BW Process (`.bwp`)** | `createBWProcess` |
| Sub-flow, sub-route, helper flow, called process | **BW SubProcess (`.bwp`, invoked via Call Process)** | `createBWProcess` + `createActivity` (Call Process) |
| Config property, app-level setting, environment variable, external config | **Module Property** (only environment-editable surface) | `createModuleProperty` |
| Secret, password, credential, API key, token | **Module Property with datatype `password`** | `createModuleProperty` (type=`password`) |
| Workflow-local input, process input, per-flow parameter | **Process Property bound to a Module Property** | `createProcessProperty` + `configureActivityAttrWithProperty` |
| Reusable resource / connection pool, JDBC DataSource, JMS ConnectionFactory | **Shared Resource** (JDBC / JMS / HTTP Client / HTTP Connector / JNDI) | `createJDBCConnection` / `createJMSConnection` / `createHttpConnectorSharedResource` / `createHttpClientSharedResource` / `createJNDIConnection` |
| Bind connection field to config value | **Shared Resource attribute binding** to a Module Property | `bindSharedResourceAttributeToProperty` |
| Group of related properties | **Module Property Group** | property-group creation via `createModuleProperty` grouping args |
| Schema, DTO, message contract, POJO | **XSD file** (referenced by activities and mappings) | filesystem write; referenced by `configureActivityWithNodeValue` |
| Service contract (SOAP), WSDL | **WSDL file** driving SOAP Receive / Reply | filesystem write; auto-consumed by `createActivity` for SOAP |
| API contract (REST), OpenAPI, Swagger | **Swagger 2.0 doc** on the REST binding | `createSwaggerOperation` / configure via `configureActivityWithValue` |

## 2. Activities (palette items) — "steps" in the spec become these

| Plain English step | BW6 palette activity | Palette | Created with |
| :--- | :--- | :--- | :--- |
| Scheduled trigger, cron job, timer starter, "every N minutes" | **Timer** (starter) | `bw.generalactivities` | `createActivity` |
| Read a DB row, SELECT, query the database | **JDBC Query** | `bw.jdbc` | `createActivity` |
| Update / insert / delete DB row | **JDBC Update** | `bw.jdbc` | `createActivity` |
| Call a stored procedure | **JDBC Call Procedure** | `bw.jdbc` | `createActivity` |
| Write / append to a file | **Write File** | `bw.file` | `createActivity` |
| Read a file | **Read File** | `bw.file` | `createActivity` |
| Watch a folder / file poller | **File Poller** (starter) | `bw.file` | `createActivity` |
| List directory / check file exists | **List Files** | `bw.file` | `createActivity` |
| Log message, console print, audit line | **Log** | `bw.generalactivities` | `createActivity` |
| Map, transform, translate, "shape the payload" | **Mapper** | `bw.generalactivities` | `createActivity` |
| Call another process / sub-flow | **Call Process** | `bw.generalactivities` | `createActivity` |
| Sleep / pause | **Sleep** | `bw.generalactivities` | `createActivity` |
| Publish to JMS / EMS topic or queue | **Send JMS Message** | `bw.jms` | `createActivity` |
| Consume from queue, JMS listener, message-driven starter | **JMS Receive Message** (starter) | `bw.jms` | `createActivity` |
| Request-reply over JMS | **Reply to JMS Message** | `bw.jms` | `createActivity` |
| REST service / HTTP endpoint / expose an API | **REST Service** (binding + Receive Request / Reply) | `bw.restjson` | `createSwaggerOperation` / `createBinding` + `createActivity` |
| Call an external REST API | **Invoke REST API** / HTTP Send Request | `bw.restjson` / `bw.http` | `createActivity` |
| SOAP service | **SOAP Receive / SOAP Reply** (from WSDL) | `bw.soap` | `createActivity` |
| Call a SOAP endpoint | **SOAP Request Reply** | `bw.soap` | `createActivity` |
| Publish/subscribe over EMS | **Publish to EMS Topic** / **Subscribe to EMS Topic** | `bw.jms` | `createActivity` |
| Branch / if-else / decision | **Choice** (with Otherwise for the fallback branch) | process transitions | `createLink` (with condition) |
| Loop / for-each / iterate | **ForEach group** | process groups | group creation via `createGroup` |
| Try/catch, error boundary | **Scope with Fault Handler** | process structure | group creation |
| Wire A to B, "linked in sequence" | **Transition** between activities | process links | `createLink` |
| End of process, terminate | **End** event | process structure | `createActivity` (End) |

## 3. Configuration verbs — how spec sentences translate to tool calls

| Spec phrasing you'll see | Meaning in BW6 | Tool |
| :--- | :--- | :--- |
| "Attribute X ← literal value" | Set a General-tab attribute to a fixed value | `configureActivityWithValue` |
| "Attribute X ← property Y" | Bind an activity attribute to a Module or Process Property | `configureActivityAttrWithProperty` |
| "Connection: `<ResourceName>`" | Set the activity's Shared Resource attribute (JDBC/JMS/HTTP) | `configureActivityWithValue` |
| "SQL: `<statement>`" | Set the `sqlStatement` attribute on JDBC Query/Update | `configureActivityWithValue` |
| "Parameter: `name` (TYPE)" | Add a prepared-statement parameter to a JDBC activity | `addJdbcParameter` |
| "Input mapping: node ← XPath expr" | Set an Input-tab node to an XPath expression | `configureActivityWithNodeValue` |
| "Input mapping: input ← whole record from prior activity" | Auto-map source output to input | `configureActivity` |
| "Compiles with zero errors" / "validate" | Run project validation | `getCompilationErrors` |
| "Export EAR" / "package for deploy" | Build the deployable archive | `exportToEAR` |

## 4. Value / datatype conventions

- `$ActivityName` in an XPath expression = reference to the **output of that activity** in the process. Keep the leading `$` and the activity name **exactly** as declared (case matters).
- `$ProcessProperty` = reference to a Process Property. Never rewrite as `$env` / `${...}` / etc.
- String literals inside XPath mappings must be **double-quoted**: `"Completed"`, not `'Completed'` or bare `Completed`.
- Datatype aliases used loosely by users → BW6 names:
  - `secret`, `credential`, `password-field` → **`password`**
  - `integer`, `int`, `int32`, `long` (non-huge) → **`Integer`**
  - `string`, `text`, `varchar` → **`String`**
  - `bool`, `boolean`, `flag` → **`Boolean`**
  - `db-connection-ref`, `datasource-ref` → **Module Property of JDBC type** (points at a Shared Resource)
  - `queue-connection-ref` → **Module Property of JMS type**

## 5. Rival-platform term normalisation (quick map)

| MuleSoft / Boomi / Camel / Spring / AWS term | BW6 equivalent |
| :--- | :--- |
| Mule flow / Camel route | BW Process (`.bwp`) |
| Mule sub-flow / Camel direct route | BW SubProcess (Call Process) |
| Mule Global Element / Boomi Connection | Shared Resource |
| Mule Configuration Property / Spring `@Value` | Module Property |
| Mule secure properties / AWS Secrets Manager entry | Module Property with datatype `password` |
| Mule DataWeave / Camel `.transform()` / Boomi Map | Mapper activity (XSLT/XPath under the hood) |
| Mule Scheduler / Spring `@Scheduled` / cron Lambda | Timer (starter) |
| Kafka consumer / SQS listener / RabbitMQ MessageListener | JMS Receive Message (BW is JMS/EMS-native; Kafka needs the Kafka palette instead) |
| Mule HTTP Listener / Spring `@RestController` | REST Service binding + Receive Request |
| Mule HTTP Request / RestTemplate / `fetch()` | Invoke REST API / HTTP Send Request |
| CXF `@WebService` / JAX-WS endpoint | SOAP Service (WSDL-driven) |
| Log4j / SLF4J logger call | Log activity |
| Camel `.choice().when()` | Choice + Otherwise transitions |
| Camel `.split()` / Mule `<foreach>` | ForEach group |
| Camel `.onException()` / Mule Try scope | Scope with Fault Handler |

## 6. Naming conventions to enforce during translation

When rewriting the prompt, apply these BW6 defaults unless the user has explicitly specified otherwise:

- **Activity names** mirror step titles the user gave, but stripped to CamelCase: `Timer`, `JDBCQuery`, `JDBCUpdate`, `WriteFile`, `Log`. Downstream XPath expressions (`$JDBCQuery/Record`, `$JDBCUpdate/noOfUpdates`) depend on these exact names.
- **Process file**: default to `Process.bwp` unless the domain suggests a better name (e.g. `RetailOrderLogger.bwp`).
- **Shared Resource**: `<Tech>Connection_<Target>` — e.g. `JDBCConnection_PostgreSQL`, `JMSConnection_EMS`.
- **Module Property names**: `UPPER_SNAKE_CASE` for values (`USERNAME`, `OUTPUT_FILE`), lowerCamel for secrets and refs when the spec says so (`jdbc_PASSWORD`, `JDBCConnectionResource`).

## 7. What to hand off after translation

Once the prompt is restated in BW6 terms, invoke the appropriate downstream skill:

- **Generic build-from-spec** → `bw6design` (drives `bwdesign` CLI or `mcp__bw__*` MCP tools).
- **User picked a canned sample** → the matching skill from `bw6-prompt-library` (e.g. `jdbcRetailInventoryDb`, `restRetailProductService`).
- **Pattern-shaped request** (REST→DB, REST→EMS, SOAP↔REST mediation, EMS subscribe, etc.) → the matching `pattern-*` skill.
- **Review / lint of an existing project** → `bw6-rules`.

Never build directly from this skill — its sole output is a normalised BW6-language prompt plus a pointer to the right builder.
