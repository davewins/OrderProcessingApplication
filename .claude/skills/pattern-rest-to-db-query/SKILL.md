---
name: pattern-rest-to-db-query
description: BW6 integration pattern — expose a REST GET that runs a JDBC query and returns records as JSON. Use when the user is designing a read-only API over a database (lookup, search, list endpoints). Trigger on phrases like "REST GET", "database read API", "JDBC query", "expose table as API", "search endpoint".
---

# Pattern: REST → Database Query

## Intent
Serve HTTP GET requests by running a **parameterized JDBC SELECT** and shaping the result set into JSON. Read-only, stateless, cacheable.

## Typical flow
```
Client → REST Binding (GET) → Mapper (query params → SQL params) → JDBC Query → Mapper (rows → JSON) → REST Reply (200)
```

## Primary use case
Retrieve records through an API. Common variants:
- Single-record lookup: `GET /customers/{id}`.
- Filtered list: `GET /orders?status=open&since=2026-01-01`.
- Cross-table read: query joins several tables into one JSON document.

## BW6 implementation

### Palette activities
| Step | Activity (palette) | Notes |
|---|---|---|
| Receive REST | **REST Service Binding** (GET operation) | Path params / query params surface as input schema. |
| Bind params | **Mapper** on the JDBC Query input | Map path/query params to `?` placeholders. |
| Query | **JDBC → JDBC Query** | Uses a **JDBC Connection** shared resource. Set the `Prepared` SQL and a *return column* schema. |
| Shape response | **Mapper** on the reply | Row list → JSON array. Apply pagination envelope if needed. |
| Reply | REST binding reply | 200 with body; 404 if lookup-by-id returned zero rows. |

### Shared resources
- **JDBC Connection** — driver, URL, credentials all bound to module properties (`bw6-rules/JDBCHardCoded.md`, `bw6-rules/BwSharedResourceUsingModuleProperty.md`).
- **HTTP Connector** with TLS on.
- Optional: **JDBC Connection Pool** tuning — check `bw6-rules/ThreadpoolUsageInJDBCActivities.md`.

### Error handling
- SQL exceptions → **Catch** → return `500` with a normalized error body (do not leak SQL error text or table names to clients).
- Empty result on a by-id lookup → return `404`, not `200` with an empty body.
- Do **not** run this in a transaction — reads don't need one, and it wastes a pool slot (`bw6-rules/CheckpointProcessJDBC.md`, `bw6-rules/JDBCTransactionParallelFlow.md`).

### Query hygiene
- **Always** use bind variables (`?`), never string-concatenate query params into SQL — SQL injection.
- Avoid `SELECT *`; enumerate columns so the return schema is stable.
- No leading `%` on `LIKE` unless indexed (`bw6-rules/JDBCWildcards.md`).

## Key validation question
> **Are pagination, sorting, and maximum result size required?**

For anything that can return more than one row, this is almost always **yes**. Decide:

1. **Max page size** (hard cap in SQL — `LIMIT`/`FETCH FIRST`) to protect the DB and the API from a runaway query.
2. **Pagination style** — offset/limit (simple, but drifts on writes) vs cursor/keyset (stable, but requires a monotonic sort key). Prefer cursor for large tables.
3. **Sort keys** — whitelist allowed `sort=` values; never interpolate them into SQL raw.
4. **Response envelope** — `{ "data": [...], "next": "..." }` vs bare array. Bare arrays are hard to evolve; envelope is recommended.

Ask the user before writing the query.

## Design checklist
- [ ] JDBC connection driver/URL/user/password bound to module properties.
- [ ] All parameters use `?` placeholders — no string concatenation.
- [ ] `SELECT` enumerates columns; return schema matches.
- [ ] Hard `LIMIT` (or `FETCH FIRST`) in the SQL, even if the API lets the caller ask for more.
- [ ] By-id lookups return `404` on empty result.
- [ ] Errors mapped to a normalized API error body (no raw SQLState leakage).
- [ ] No transaction wrapper on a pure read.
- [ ] Tests cover: happy path, not-found, injection-attempt input, oversized page request.

## Authoring in BW6
Use [[bw6design]] to add the REST service binding, `JDBC Query` activity, and shared resource. Apply [[bw6-rules]] rules — especially `JDBCHardCoded.md`, `JDBCWildcards.md`, `ThreadpoolUsageInJDBCActivities.md`.
