---
name: pattern-rest-to-db-command
description: BW6 integration pattern — expose a REST POST/PUT/DELETE that performs a JDBC insert/update/delete inside a transaction and returns the result. Use when the user is designing a write-side API over a database (create, update, upsert, soft-delete endpoints). Trigger on phrases like "REST POST/PUT", "write API", "JDBC insert", "update endpoint", "delete record".
---

# Pattern: REST → Database Command

## Intent
Serve HTTP write requests (POST / PUT / PATCH / DELETE) by executing a **JDBC INSERT / UPDATE / DELETE / MERGE** and returning the outcome. Transactional, non-idempotent by default — safety is your responsibility.

## Typical flow
```
Client → REST Binding (POST/PUT/DELETE) → Mapper (payload → SQL params) → [Transaction Group] → JDBC Update → REST Reply (201/200/204)
                                                                                    │
                                                                                    └── Catch → rollback + normalized error
```

## Primary use case
Create or update database records via API. Common variants:
- `POST /orders` — insert, return `201 Created` with location header.
- `PUT /customers/{id}` — full update / upsert.
- `PATCH /customers/{id}` — partial update.
- `DELETE /orders/{id}` — hard delete or soft-delete (`deleted_at = now()`).

## BW6 implementation

### Palette activities
| Step | Activity (palette) | Notes |
|---|---|---|
| Receive REST | **REST Service Binding** (POST/PUT/PATCH/DELETE) | Validate the JSON body against the operation schema. |
| Transaction | **General Activities → Group** with `Group Type = Transaction Group` (or start a JDBC transaction on the connection) | Ensures all writes commit or rollback together. |
| Write | **JDBC → JDBC Update** (or **JDBC Call Procedure**) | For multi-row writes, use `JDBC Update` in a loop **inside** the transaction group. |
| Reply | REST binding reply | `201` on create with `Location: /resource/{id}`; `200` on update with body; `204` on delete. |

### Shared resources
- **JDBC Connection** — driver/URL/creds bound to module properties.
- **HTTP Connector** with TLS (`bw6-rules/SSLServerConnectorShouldHaveTLSprotocol.md`).

### Transaction rules
- All DB modifications for one request go inside **one** transaction group.
- On any error inside the group, the Catch handler must let the group **rollback** (do not swallow the fault before it exits the group boundary).
- Do not mix a JDBC transaction with a JMS Send in the same group unless you deliberately want XA — see `bw6-rules/CheckpointProcessTransaction.md` and `bw6-rules/JDBCTransactionParallelFlow.md`.
- **Never** call an external HTTP service from inside the transaction group unless you have a compensating action; the transaction can hold the DB row lock while a slow HTTP call is pending.

### Idempotency
- POST is not idempotent by default. If the client may retry, accept an `Idempotency-Key` header and dedupe on it (unique index on the key).
- PUT / DELETE should be idempotent — write the SQL so a repeat call has no additional effect (`UPDATE ... WHERE id = ?`, `DELETE ... WHERE id = ?`).

### Error mapping
- Unique constraint / FK violation → `409 Conflict`.
- Not found on `PUT`/`DELETE` (no rows affected) → `404`, not `200`.
- All other SQL errors → `500` with a normalized body — do not leak SQLState or table names.

## Key validation question
> **Is the API responsible for transaction control?**

Two variants — pick one:

1. **API owns the transaction (default).** BW6 opens the transaction on entry, commits on the success reply, rolls back on any fault. This is the simple, correct default for the vast majority of write endpoints.
2. **API delegates to a stored procedure / DB-side transaction.** The proc opens and closes its own transaction; BW6 just calls it and translates the return status. Use when the SQL logic is complex, spans many tables, or already exists as a proc. BW6 must **not** open a second transaction on top.

Ask the user which model applies before wiring the group.

## Design checklist
- [ ] JDBC connection driver/URL/user/password bound to module properties (`bw6-rules/JDBCHardCoded.md`).
- [ ] All parameters use `?` placeholders — no string concatenation into SQL.
- [ ] Writes are inside a Transaction Group with a Catch that lets it rollback.
- [ ] `POST` returns `201 Created` with a `Location` header.
- [ ] `PUT`/`DELETE` return `404` when no rows are affected.
- [ ] Idempotency strategy (idempotency key, or naturally-idempotent SQL) is documented.
- [ ] No external HTTP calls inside the transaction group.
- [ ] Unique-constraint failures mapped to `409`, not `500`.
- [ ] Tests cover: happy path, duplicate insert, update-of-missing-id, invalid body, DB down.

## Authoring in BW6
Use [[bw6design]] to add the REST service, the Transaction Group, the `JDBC Update` activity, and the reply mapper. Apply [[bw6-rules]] — especially `JDBCHardCoded.md`, `CheckpointProcessTransaction.md`, `JDBCTransactionParallelFlow.md`, `ExceptionHandlingCheck.md`.
