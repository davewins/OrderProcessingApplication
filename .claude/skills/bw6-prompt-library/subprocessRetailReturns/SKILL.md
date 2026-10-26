---
name: subprocessRetailReturns
description: Build the "retail.bw.sample.palette.subprocess.RetailReturnProcess" BW6 application — a Timer-driven main process that delegates return validation to a subprocess via CallProcess. Use when the user asks to create/scaffold the retail returns subprocess example, needs a BW6 SubProcess palette sample showing main-process → subprocess handoff with Mapper, or references any of: "retail returns", "RetailReturnProcess", "ReturnMainProcess.bwp", "ValidateReturn.bwp", "retail return subprocess", "CallProcess sample". Drives the build via `bwdesign` / `mcp__bw__*` tools and applies `bw6-rules` checks.
---

# retail.bw.sample.palette.subprocess.RetailReturnProcess — SubProcess Sample (BW6)

Demonstrates the **main-process → subprocess** call pattern with a Timer starter, `CallProcess`, and a validation subprocess that maps and logs return status.

Category: **SubProcess** • Main tech: `Timer, Call Process, Mapper`.

## How to run this skill

1. Confirm the target BW workspace with the user. Prefer `mcp__bw__*` if Business Studio is open, else `bwdesign` per the `bwdesign` skill.
2. Execute the spec below step by step. Announce each major step.
3. Cross-check against `bw6-rules`. Rules to watch:
   - `SubProcessInlineCheck` — if the caller is later refactored to Inline, move any large payload into a Job Shared Variable.
   - `LogSubprocess` — this template already puts logging inside a subprocess step, good.
   - `LastActivityAndEndActivity` — `ValidateReturn.bwp` correctly ends with `End`.
   - `ProcessNoDescription`, `ProcessNamingConvention` — fill descriptions.
   - `AtLeastOneStarter` — the Timer in `ReturnMainProcess.bwp` satisfies this.
4. Validate and report status.

## Project Specification

### Project Hierarchy

| Component Type | Name |
| :---- | :---- |
| **Application Module** | `retail.bw.sample.palette.subprocess.RetailReturnProcess` |
| **Application Project** | `retail.bw.sample.palette.subprocess.RetailReturnProcess.application` |

### Process Architecture

| Package Name | Process Name |
| :---- | :---- |
| `retailreturnprocess` | `ReturnMainProcess.bwp` |
| `retailreturnprocess` | `ValidateReturn.bwp` |

### `ReturnMainProcess.bwp`

Activities: `Timer` → `Log` → `CallProcess` → `Log1`. Link in sequence.

- **Timer (Starter)** — `Interval` = `60` seconds.
- **Log** — `message` = `"Retail return initiated"`
- **CallProcess** — `Process Name` = `ValidateReturn.bwp`
- **Log1** — `message` = `"Retail return completed"`

### `ValidateReturn.bwp`

Activities: `Start` → `Mapper` → `Log` → `End`. Link in sequence.

- **Start**.
- **Mapper**
  - `customerId` ← `"CUST1001"` (ensure `$customerId` process variable exists)
  - `returnStatus` ← `"Approved"` (ensure `$returnStatus` process variable exists)
- **Log** — `message` = `$Mapper/returnStatus`
- **End**.
