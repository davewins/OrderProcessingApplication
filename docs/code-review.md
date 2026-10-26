# BW6 Code Review — OrderProcessing Module

**Date:** 2026-10-26
**Scope:** `OrderProcessing.module` (11 BW processes, shared resources, module config)
**Method:** Reviewed against the BW6 quality ruleset (`bw6-rules`), ranked by severity.
**Remediation (2026-10-26):** #1 and #5 fixed via the `bwdesign` CLI (module validates clean); #6 blocked by tooling; #2/#3/#4/#7 deferred by decision. See per-finding status below.

> **Context:** This is a *simulation/demo* application — backend database work is faked with
> randomized sleep timers. Some findings below are intentional for showcase purposes. However,
> the credential and transport-security items are worth fixing regardless of demo status, since
> the project is meant to demonstrate real-world best practices on the TIBCO Platform.

---

## Summary

| # | Finding | Rule | Severity | Status |
|---|---------|------|----------|--------|
| 1 | Committed EMS credentials in `default.substvar` | — (secret hygiene) | 🔴 High | ✅ Fixed |
| 2 | Inbound REST binding has no security policy | `BindingShouldHavePolicyAssociated` | 🔴 High | ⏸ Deferred |
| 3 | HTTP connector is plaintext (no TLS) | `HttpConnectorShouldHaveConfidentiality` | 🟠 Medium | ⏸ Deferred |
| 4 | JMS connection is non-SSL | `JMSConnectorShouldHaveConfidentiality` | 🟠 Medium | ⏸ Deferred |
| 5 | Hardcoded JMS destinations | `JMSHardCoded` | 🟠 Medium | ✅ Fixed |
| 6 | No process descriptions | `ProcessNoDescription` | 🟡 Low | ⛔ Blocked (tooling) |
| 7 | No real unit tests | `ProcessWithoutTest` | 🟡 Low | ⏸ Deferred |

---

## Security — highest priority

### 1. Committed EMS credentials in `default.substvar` 🔴
**File:** `OrderProcessing.module/META-INF/default.substvar`

The module default profile ships live-looking connection secrets committed to git:

- `EMSUserName = admin`
- `EMSPassword = #!^…` (redacted here) — BW-obfuscated, **not** encrypted (trivially reversible)
- `EMSURL = tcp://192.168.1.30:9011` — hardcodes a real private-network host

**Recommendation:** Blank these defaults (or use obvious placeholders) and supply the real values
at deploy time via TIBCO Platform application properties. Rotate the committed password.

> **✅ Fixed (2026-10-26, `bwdesign`):** `EMSUserName` and `EMSURL` defaults blanked; `EMSPassword`
> cleared (the field now holds BW's obfuscation of an empty string — the real secret is removed).
> The secret remains in prior git history, so **rotate it** if it was ever real.

### 2. Inbound REST binding has no security policy 🔴
**File:** `OrderProcessing.module/META-INF/module.bwm` · **Rule:** `BindingShouldHavePolicyAssociated`

Three `rest:RestServiceBinding` endpoints are exposed with no policy associated, and the
`Policies/` directory is empty:

- `POST /api/orders`
- `GET` / `PUT` `/api/orders/{orderId}`
- `GET /api/orders/customer/{customerId}`

All public endpoints are therefore unauthenticated.

**Recommendation:** Associate an authentication policy with each inbound binding (and prefer a
secure scheme over HTTP Basic — see `BindingShouldNotHaveHTTPBasicPolicyAssociated`).

### 3. HTTP connector is plaintext (no TLS) 🟠
**File:** `OrderProcessing.module/Resources/orderprocessing/module/api.httpConnResource`
**Rule:** `HttpConnectorShouldHaveConfidentiality`

The HTTP connector has no TLS/SSL/confidentiality configuration — the API serves over plain HTTP.

> **Config inconsistency:** the Swagger/README advertise port **8080**, but the connector binds the
> `Port` property = **8282** while `BW.CLOUD.PORT` = 8080. Reconcile these.

### 4. JMS connection is non-SSL 🟠
**File:** `OrderProcessing.module/Resources/orderprocessing/module/JMSConnectionResource.jmsConnResource`
**Rule:** `JMSConnectorShouldHaveConfidentiality`

The connection factory uses `tcp://` (via `EMSURL`) with no SSL settings, so all EMS traffic
(orders, audit logs) is unencrypted.

---

## Config hygiene

### 5. Hardcoded JMS destinations 🟠
**Rule:** `JMSHardCoded`

Queue names are literals inside the activities instead of module properties:

| Process | Location | Queue |
|---------|----------|-------|
| `CreateOrderProcess.bwp` | line ~1404 | `Orders` |
| `AuditLog.bwp` | line ~609 | `LogMessage` |
| `StockCheck.bwp` | — | `StockCheck` |

**Recommendation:** Move queue names to module properties so they are environment-configurable.

> **✅ Fixed (2026-10-26, `bwdesign`):** Added string module properties `OrdersQueue` (`Orders`),
> `LogMessageQueue` (`LogMessage`), `StockCheckQueue` (`StockCheck`) and bound each JMS activity's
> `destinationName` to its property via `attributeBindings`. Module validates clean.

> **Good, for contrast:** the HTTP endpoint host/port *are* property-driven (`BW.HOST.NAME`, `Port`),
> satisfying `EndpointURIFromHTTPBindingSetUsingProperty`, and the JMS provider URL/credentials are
> property-bound in the shared resource.

---

## Process quality

### 6. No process descriptions 🟡
**Rule:** `ProcessNoDescription`

All 11 processes have an empty `description=""`.

> **⛔ Blocked (2026-10-26):** `bwdesign` exposes no command to set a process description, and the
> sanctioned fallback (BW Studio MCP server) is disabled by enterprise policy on this machine.
> Per `AGENTS.md` the `.bwp` files must not be hand-edited, so this is left for Studio or a future
> MCP-enabled session.

### 7. No real unit tests 🟡
**Rule:** `ProcessWithoutTest`

`OrderProcessing.module/Tests/A3DEWS2RF4.ml` is an empty `BWTFile` stub (no test cases) and is the
only test for 11 processes. Every process is effectively untested.

---

## Checked and compliant (no finding)

- **Checkpoint placement** (`CheckpointProcessJDBC` / `HTTP` / `REST`) — no Checkpoint activities
  present, so these rules do not apply.
- **`JMSReceiverPlusConfirm`** — `StockCheck` uses *Explicit Client* acknowledgement **and** has a
  matching `bw.generalactivities.confirm` activity on its success path. Compliant.
- **`LogSubprocess`** — Log activities appear only in the `Logger` subprocess, not in component
  processes. Compliant.
- **`ProcessNamingConvention`** — all processes use consistent `orderprocessing.module.*` names.

---

## Suggested remediation order

1. Blank the credential defaults and rotate the committed EMS password (#1).
2. Add authentication policies to the REST bindings (#2).
3. Enable TLS on the HTTP connector and SSL on the JMS connection (#3, #4).
4. Parameterise JMS destinations via module properties (#5).
5. Backfill process descriptions and real unit tests (#6, #7).

> **Note:** All BW source edits must go through the `bwdesign` CLI (or the BW Studio MCP server) —
> never hand-edit `.bwp`, `module.bwm`, `.substvar`, or shared-resource files directly.
> See `AGENTS.md`.
