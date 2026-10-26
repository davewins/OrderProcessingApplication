# AGENTS.md — Building TIBCO BusinessWorks 6 Applications

Instructions for any AI agent working in this repository. Read this **before** touching a BW project.

---

## ⛔ THE HARD RULE: NEVER DIRECTLY EDIT BW SOURCE FILES

**Do not create, edit, patch, or delete BW project files with `Write`, `Edit`, `sed`, `awk`, `python`, or any other direct filesystem tool.**

This applies to every file below:

| File / folder | What it is |
|---|---|
| `Processes/**/*.bwp` | BW processes — the flow logic |
| `META-INF/module.bwm` | The module model |
| `META-INF/MANIFEST.MF` | OSGi + `TIBCO-BW-*` headers (module identity, version, palettes) |
| `META-INF/default.substvar` | Config profile / substitution variables |
| `META-INF/module.jsv`, `module.msv` | Job / module shared variables |
| `Resources/**` | Shared resources (HTTP, JMS, JNDI, JDBC, SSL, …) |
| `Service Descriptors/**` | WSDLs |
| `Schemas/**` | XSDs |
| `Policies/**` | Policy files |
| `Tests/**/*.bwt`, `*.bwts` | Test files and suites |
| `*.application/META-INF/**` | Application wrapper + `TIBCO.xml` |

**Always author BW artifacts through one of exactly two interfaces:**

1. The **`bwdesign` CLI** (default — see below), or
2. The **BW Studio MCP server** (fallback — see below).

### Why this rule exists

A `.bwp` is not hand-authorable XML. It carries four coupled representations of the same process that must stay in sync:

- the BPEL model (`bpws:*` activities, links, scopes),
- the GMF **diagram notation** block (`notation:Diagram`, layout constraints, per-activity children nodes),
- the **variable descriptor** and `bpws:variables` typing,
- per-activity **XSLT input bindings**, whose template names must match the activity's input variable.

Editing the XML by hand desynchronises these. The usual results are an activity that executes but is invisible in the Studio canvas, a process that silently loses its diagram, or a corrupt file that Studio refuses to open. The CLI and MCP server drive Studio's own model APIs and update all four representations together.

There is also a **live-state hazard**: when Studio has the project open it holds its own in-memory copy. A write to disk behind its back is either ignored or clobbered on the next Studio save — and you will not be told.

### The narrow exception, and its cost

The MCP server exposes generic file tools (`replaceString`, `applyPatch`, `insertIntoFile`, `replaceFileContent`, `createFile`). These route through the Eclipse resource layer, so they are **safe with respect to the lock and to Studio's in-memory state** — but they still write raw XML and therefore still bypass the model APIs.

Treat them as a last resort. Before using one:

1. Confirm no modelling tool does the job (`configureActivityWithNodeValue`, `configureActivityWithValue`, `configureActivity`, `configureActivityEditorSchema`, `configureActivityAttrWithProperty`, …).
2. Keep the edit surgical — a namespace declaration, a stale template name. Never structural changes (adding activities, links, or variables).
3. Re-run validation immediately afterwards (see *Verification*).
4. Say in your response that you did it and why.

Creating a **new** standalone XSD or WSDL is the one routine case where a file tool is legitimate — there is no `createSchema` command. Use MCP `createFile` when Studio is open; a plain `Write` is acceptable only when the workspace is closed and no BW model file is being modified.

---

## Environment

These skills make no assumption about where BW is installed. Resolve the three values below at the start of a session — from the project's own `CLAUDE.md`/`AGENTS.md` if it pins them, otherwise by asking the user.

```
BW_HOME    # BW install root — the directory containing bw/<version>/
BW_VERSION # e.g. 6.12
bwdesign   = $BW_HOME/bw/$BW_VERSION/bin/bwdesign
WS         # Eclipse workspace holding the BW projects
```

If `BW_HOME` is unknown, the install root is the parent of a `bw/<version>/bin/bwdesign` path:

```bash
find "$HOME" -type f -name bwdesign -path '*/bw/*/bin/*' 2>/dev/null
```

Confirm the build before relying on any version-specific behaviour — the gotchas below were verified against **6.12.0 HF3**:

```bash
cd "$BW_HOME/bw/$BW_VERSION/bin" && printf 'edition\nexit\n' | ./bwdesign -data "$WS" -silent
```

**Reference workspace.** Several workarounds below read activity type IDs out of an existing workspace of worked examples. Any workspace containing real `.bwp` files serves; substitute its path for `<REFERENCE_WS>` wherever it appears.

---

## Which interface to use — decision procedure

```
                    ┌─────────────────────────────┐
                    │ Need to author a BW artifact│
                    └──────────────┬──────────────┘
                                   ▼
                    ┌─────────────────────────────┐
                 1. │  Try bwdesign CLI (DEFAULT) │
                    └──────────────┬──────────────┘
                                   │
        "Workspace [...] is currently in use by another application."
                                   ▼
                    ┌─────────────────────────────┐
                 2. │  Fall back to the MCP server│
                    └──────────────┬──────────────┘
                                   │
                       connection refused / no MCP response
                                   ▼
                    ┌─────────────────────────────┐
                 3. │  STOP. Ask the user to      │
                    │  enable it in Studio.       │
                    └─────────────────────────────┘
```

**Never** resolve a locked workspace by editing files directly, by killing Studio, or by deleting `.metadata/.lock`. Steps 2 and 3 are the only paths.

---

## 1. Default path — the `bwdesign` CLI

Preferred because it is deterministic, scriptable, and works without Studio running.

```bash
cd "$BW_HOME/bw/6.12/bin"          # MUST cd here first
printf 'system:createBWApplicationModule MyApp MyMainProcess 1.0.0\nexit\n' \
  | timeout 300 ./bwdesign -data "$WS" -silent 2>&1 | sed 's/\x1b\[[0-9;]*m//g'
```

### Non-negotiable mechanics

- **Run from `bin/`.** The wrapper reads `bwdesign.tra` from the cwd; an absolute path from elsewhere fails with `Failed to open properties file : bwdesign.tra`.
- **Always pass `-data <workspace>`.** The default workspace may be locked by Studio and will crash the JVM.
- **Pipe commands on stdin, one per line, ending with `exit`.**
- **Strip ANSI codes**: `sed 's/\x1b\[[0-9;]*m//g'`.
- **Wrap in `timeout`** and keep batches small — one hung command blocks the entire batch. Budget ~40 s of JVM startup per invocation.
- **`help` takes the bare command name**: `help createProcess`, *not* `help system:createProcess`.

### Verified gotchas (BW 6.12.0 HF3)

- **`system:listActivityTypes` prints nothing.** Confirmed broken, both piped and as a direct argument. Harvest the IDs from any workspace of existing BW projects instead (recurse — processes nest below the package folder):
  ```bash
  find <REFERENCE_WS> -name '*.bwp' -exec grep -ohE 'activityTypeID="[^"]*"' {} + | sort -u
  ```
  Common IDs: `bw.generalactivities.mapper`, `bw.generalactivities.log`, `bw.generalactivities.timer`, `bw.generalactivities.callprocess`. If the MCP server is reachable, its `ListActivity` tool returns the full palette and is the better source.
- **`system:configureActivity -nv` writes literals only.** It wraps the value in XPath quotes: `select="&quot;Hello World&quot;"`. It cannot produce a reference to another activity's output.
- **`system:configureActivity -n` (source-node mapping) is effectively unusable** — every argument arity tried returns `Insufficient arguments for Node Mapping!`. **If you need a real activity-to-activity mapping, that is a reason to use the MCP server** (`configureActivityWithNodeValue` accepts `$Activity/field`), not a reason to edit the `.bwp`.
- **`system:configureActivityEditorSchema` only works on activities with an Input/Output Editor tab** (Render JSON, Parse JSON). For a Mapper, use `system:configureInputVariable` instead — it types both the input and output variables.
- The message `trying to replace attribute on null element: ... ActivityExtensionImpl` on `createActivity` is **benign noise**; check the resulting model, not this line.

Full command catalog: `.claude/skills/bwdesign/reference.md`.

---

## 2. Fallback path — the BW Studio MCP server

Use **only** when the CLI reports the workspace is in use. The server runs inside Studio, so it operates on **whatever workspace Studio currently has open** — there is no `-data` equivalent. Confirm the target before writing:

```
mcp__bw__listBWProjects        → project names + on-disk locations
mcp__bw__getAllProjectsInWorkspace
```

### Connecting

Configured in `.mcp.json` at the repo root as the server named **`bw`**, so its tools appear as `mcp__bw__<toolName>`. That name is deliberate — the `glossary` skill instructs downstream steps to use `mcp__bw__*`.

Project-scoped `.mcp.json` servers need a **one-time approval** the first time Claude Code starts in this directory.

**The port is not fixed.** It is a Studio preference and has been observed as 8181 and 8686; `system:setHttpMcpServerEnabled` documents 8080 as its default. Do not assume — discover it:

```bash
for p in 8080 8181 8686; do
  printf "%s: " "$p"
  curl -s -m 3 -X POST "http://localhost:$p/mcp" \
    -H 'Content-Type: application/json' \
    -H 'Accept: application/json, text/event-stream' \
    -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"probe","version":"1"}}}' \
    | head -c 120; echo
done
```

A live BW server answers with `"serverInfo":{"name":"BW Copilot",...}`. **Anything returning HTML is a different application** — an HTTP 200 alone does not mean you found it. Then update the `url` in `.mcp.json` to match.

### Protocol quirk

Streamable HTTP. `initialize` answers inline on the POST, but **`tools/list` and `tools/call` results arrive on the GET SSE stream**, not in the POST body. A POST-only client sees empty responses and looks broken. Correlate by JSON-RPC id across both channels.

### Key tools

| Task | Tool |
|---|---|
| Inspect | `listBWProjects`, `getActivitiesInProcess`, `getProcessState`, `explainProcess`, `getProjectLayout`, `readProjectResource` |
| Create | `createBWApplicationProject`, `createBWProcess`, `createSubProcess`, `createActivity`, `createActivityInGroup`, `createGroup`, `createCatch` |
| Wire | `createLink`, `createErrorLink`, `createLinkInGroup`, `renameActivity` |
| Configure | `configureActivityWithValue` (General tab, literals), `configureActivityWithNodeValue` (Input tab; accepts `$Activity/field`), `configureActivity` (map an entire activity output), `configureInputVariable`, `configureActivityEditorSchema` |
| Resources | `createHttpConnectorSharedResource`, `createJDBCConnection`, `createJMSConnection`, `createJNDIConnection`, `createSSLClientResource`, `connectResource`, `configureSharedResource` |
| Properties | `createModuleProperty`, `updateModulePropertyValue`, `createProcessProperty`, `configureActivityAttrWithProperty` |
| Verify / ship | `getCompilationErrors`, `exportToEAR`, `generatePom`, `runMavenTestGoal` |

### Known MCP quirks

- **`configureActivityWithNodeValue` does not declare namespace prefixes.** It writes the XPath verbatim into the activity's XSLT. If the source schema is `elementFormDefault="qualified"`, `$Mapper/message` fails with `No matching message` and `$Mapper/greet:message` fails because `greet` is undeclared. **Prefer designing schemas as `elementFormDefault="unqualified"`** so the generated XPaths resolve without intervention.
- **`renameActivity` leaves the XSLT template name stale** — it updates the activity name and its input variable (`more_logs-input`) but the binding template keeps the old name (`Log1-input`). Usually harmless; re-configuring the node value does not fix it.
- `createActivity` returns the BW-assigned name (e.g. `Log1`); call `renameActivity` only if you need a different one.

---

## 3. If the MCP server cannot be reached — ask the user

Do **not** work around it. Do not edit files. Do not kill Studio. Stop and ask the user to enable the server:

> The BW workspace is locked by Studio and I can't reach the Studio MCP server. Please enable it:
>
> **Window ▸ Preferences ▸ BusinessWorks AI ▸ HTTP MCP Server**
> - tick **Enable HTTP MCP Server**
> - note the **Hostname** (`localhost`) and **Port**
> - confirm **Server Status** reads *HTTP Server is running*
>
> Then tell me the port and I'll continue.

The preference page lists the enabled endpoints, e.g.:

```
http://localhost:<port>/mcp/BusinessWorks-Studio
http://localhost:<port>/mcp/web-tools
http://localhost:<port>/mcp/time
http://localhost:<port>/mcp/webpage-reader
http://localhost:<port>/mcp/tibco-pdf-search
http://localhost:<port>/mcp/memory
http://localhost:<port>/mcp/bw-documentation
http://localhost:<port>/mcp          ← unified (all servers)
```

Use the **unified `/mcp`** endpoint in `.mcp.json` — it exposes the BW design tools plus the docs/search tools in one connection.

If the workspace is *not* open in Studio, the preference can also be set headlessly (Eclipse preferences are per-workspace, so this only works on a workspace nothing else holds):

```bash
printf 'system:setHttpMcpServerEnabled true localhost 8686\nexit\n' \
  | ./bwdesign -data "$WS" -silent
```

---

## Skills

Invoke with the `Skill` tool. Read the skill before improvising.

| Skill | Use for |
|---|---|
| `bwdesign` | Full `bwdesign` command catalog and mechanics. **Read before driving the CLI.** |
| `glossary` | **Run first** on any vague ask. Translates plain-English or rival-platform vocabulary (Mule flow, Boomi process, Camel route, Lambda) into BW6 nouns and `mcp__bw__*` tool names. Does not build anything. |
| `bw6-rules` | Quality/design review: security posture, TLS, hardcoded credentials, checkpoint & transaction placement, JDBC/JMS/HTTP misuse, namespace hygiene, SonarQube-style rules. |
| `bw6-prompt-library` | Browse-only catalog of the sample-application prompt skills. |

Integration pattern skills — each scaffolds a canonical shape:

| Skill | Pattern |
|---|---|
| `pattern-rest-to-db-query` | REST GET → JDBC query → JSON |
| `pattern-rest-to-db-command` | REST POST/PUT/DELETE → transactional JDBC write |
| `pattern-rest-to-ems` | REST → canonical → EMS publish → HTTP 202 |
| `pattern-source-to-ems-publish` | DB poll / file watch / schedule → canonical → EMS (producer, CDC) |
| `pattern-ems-subscribe-to-target` | EMS subscribe → transform → DB / REST / SOAP / file / ERP (consumer) |
| `pattern-rest-to-soap-mediation` | REST/JSON front over a legacy SOAP backend |
| `pattern-soap-to-rest-mediation` | Preserve a SOAP contract over a REST backend (strangler) |
| `pattern-rest-sync-backend-proxy` | API facade / BFF over an existing REST or SOAP backend |

---

## Standard build lifecycle

1. **Clarify** — run `glossary` if the request is in generic integration terms.
2. **Scaffold** — `system:createBWApplicationModule <App> <MainProcess> 1.0.0`. Creates both `<App>/` (the module) and `<App>.application/` (the EAR wrapper). Note the main process lands in a package named after the lowercased app, so its qualified name is e.g. `helloworld.Demo`.
3. **Author** — processes, activities, links, groups, catches.
4. **Schemas / resources** — XSDs, WSDLs, shared resources, module properties.
5. **Configure** — activity General-tab attributes and Input-tab mappings.
6. **Validate** — must be clean before you report success.
7. **Package** — `system:export -ear <App> -path <dir>` or `exportToEAR`. Only when asked.

### Every process needs a starter

A process with no process starter or service fails validation:

```
TIBCO-BW-VALIDATION-500113: The process [x] must be implemented with a process starter or a process service.
```

Add a Timer, a Receive/Pick with `createInstance=true`, or a service binding. If the user's spec omits one, add the minimal starter, **and say so explicitly in your response** — it is an addition beyond what they asked for.

---

## Verification — required before reporting success

Never report a BW app as done without a clean validation run.

```bash
# CLI
printf 'system:validate <Module>\nexit\n' | ./bwdesign -data "$WS" -silent 2>&1 | sed 's/\x1b\[[0-9;]*m//g'
```

```
# MCP
mcp__bw__getCompilationErrors {"projectName": "<Module>"}
```

Empty output from `system:validate` means success. Report the actual result — if errors remain, quote them.

---

## Response conventions

- State which interface you used (CLI or MCP) and, if you fell back, why.
- Call out any addition you made beyond the literal request (a starter activity, a schema).
- If you used an MCP file tool under the narrow exception above, say so and say why no modelling tool covered it.
- Quote validation output rather than asserting "it validates".
