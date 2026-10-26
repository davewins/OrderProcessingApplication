---
name: bwdesign
description: Drive the TIBCO BusinessWorks 6.x design-time CLI (bwdesign) to author and manage BW projects from the command line — create/import/export applications and modules, add processes/activities/groups/links, configure module & process properties, shared resources (HTTP/JMS/JNDI), WSDL/SOAP/REST bindings, Swagger operations, tests & assertions, validate, generate pom.xml/diagrams/EARs, and configure MCP. Use whenever the user wants to script or automate BW6 Studio design tasks, or asks how to run bwdesign / a specific bwdesign command.
---

# bwdesign — BusinessWorks 6.x Design-Time CLI

`bwdesign` is the headless command-line front end to TIBCO BusinessWorks Studio (Eclipse-based). It opens (or creates) an Eclipse workspace and runs design-time operations against the BW projects in it.

> **`<BW_HOME>`** below is your TIBCO BusinessWorks 6.x install root (the folder that contains `bw/6.12/`). Substitute your own path.

- **Tool path:** `<BW_HOME>/bw/6.12/bin/bwdesign`
- **Version confirmed:** BW 6.12.0 HF3 (run `edition` to print the edition).
- **Full command catalog:** see [reference.md](reference.md) for syntax/arguments/options of every command.

## Running it (read this first — there are real gotchas)

1. **Run from the `bin` directory.** The wrapper needs `bwdesign.tra` in the current working directory. Running it by absolute path from elsewhere fails with `Failed to open properties file : bwdesign.tra`.
   ```bash
   cd <BW_HOME>/bw/6.12/bin
   ```

2. **Always pass an isolated `-data <workspace>`** unless you specifically need the user's Studio workspace. Without it bwdesign defaults to the platform default workspace (e.g. `<your-home>/workspace`), and if that workspace is open in the IDE the launch fails with an Eclipse `Workspace is already closed` / OSGi bundle error. A scratch dir avoids the lock:
   ```bash
   ./bwdesign -data /tmp/bwdesign_ws -silent <command>
   ```
   To operate on real projects, point `-data` at the workspace that contains them (and make sure Studio isn't holding it open).

3. **Usage form:** `bwdesign [options] command <args>`. With no command it drops into an interactive `bwdesign>` shell.

4. **Batch / non-interactive:** pipe commands on stdin, one per line, ending with `exit`:
   ```bash
   printf 'system:createProcess -p MyModule MyProc com.acme.pkg\nexit\n' | ./bwdesign -data /tmp/bwdesign_ws -silent
   ```

5. **Strip ANSI color codes** from captured output (the shell emits them even when piped):
   ```bash
   ... | sed 's/\x1b\[[0-9;]*m//g'
   ```

6. **One hung command blocks the whole batch.** When scripting many commands, prefer one invocation per logical step (or small chunks) and wrap with `timeout` so a stall doesn't hang everything:
   ```bash
   printf '<cmd>\nexit\n' | timeout 150 ./bwdesign -data /tmp/bwdesign_ws -silent 2>&1 | sed 's/\x1b\[[0-9;]*m//g'
   ```

7. **`help` syntax gotcha:** use the **bare** command name — `help createProcess`, **not** `help system:createProcess`. In batch mode the `area:command` form returns empty output (and `help area:command` for the `name:name` commands errors with `Area not found`). `help` / `help -all` lists all commands.

## Global options

| Option | Meaning |
|---|---|
| `-data <workspace>` | Eclipse workspace folder (defaults to Studio's current workspace — override it, see above) |
| `-clean` | Clean the Eclipse workspace on launch |
| `-dir <directory>` | Set the current working directory |
| `-log [-a] <file>` | Send (or append, with `-a`) output to a file |
| `-time` | Prefix log messages with timestamps |
| `-debug` | Extra logging |
| `-silent` | Suppress the intro/help banner |

## Command groups (full details in reference.md)

- **Shell/workspace:** `cd`, `pwd`, `ls`, `clear`, `clean`, `edition`, `setedition`, `execute` (run a batch script file), `exit`.
- **Projects & build:** `system:create`, `system:createBWApplicationModule`, `system:delete`, `system:import`, `system:export`, `system:importpreferences`, `system:validate`, `generatepom`, `addDependency`, `copyArtifacts`, `diagram:gen_diagrams`, `generate_manifest_json`.
- **Processes & activities:** `system:createProcess`, `system:createActivity`, `system:renameActivity`, `system:linkActivity`, `system:configureActivity`, `system:configureInputVariable`, `system:configureActivityEditorSchema`, `system:createGroup`, `system:createCatch`, `system:setCallProcessName`, `system:listActivityTypes`, `system:addtag`, `system:configuretag`, `system:bwPackage`, `system:addjdbcparam`.
- **Services/WSDL/bindings:** `system:createWSDL`, `system:setServicePortType`, `system:createBinding`, `system:setSOAPBindingTransportType`, `createSwaggerOperation`, `system:createConversation`, `system:configureConversationKey`.
- **Properties, variables & resources:** `system:moduleProperty`, `system:processProperty`, `system:processVariable`, `system:sharedVariable`, `system:display`, `system:copyModuleProp`, `system:connectResource`, `createHttpConnector`, `createJMSConnection`, `createJNDIConnection`, `system:createApplicationProfile`, `system:renameApplicationProfile`, `system:deleteApplicationProfile`, `export_to_consul`.
- **Testing:** `createTestFolder`, `createTestSuite`, `createTestFile`, `addAssertion`, `deleteAssertion`, `listAllTests`, `mockOutputFile`.
- **MCP (AI):** `generate_mcp_skeleton`, `generate_mcp_config`, `generate_mcp_ear`, `system:setHttpMcpServerEnabled`.

## Typical workflow

```bash
cd <BW_HOME>/bw/6.12/bin
WS=/path/to/workspace        # contains/will contain the BW projects
run() { printf '%s\nexit\n' "$1" | timeout 300 ./bwdesign -data "$WS" -silent 2>&1 | sed 's/\x1b\[[0-9;]*m//g'; }

# 1. Application + module + main process.
#    Creates module project `MyApp` (NOT MyApp.module) and app project `MyApp.application`.
#    The main process lands in package `myapp` → qualified name `myapp.MyMainProcess`.
run 'system:createBWApplicationModule MyApp MyMainProcess 1.0.0'

# 2. Author. Module name is the bare `MyApp`; processes take the qualified name.
run 'system:createActivity MyApp myapp.MyMainProcess bw.generalactivities.timer'
run 'system:createActivity MyApp myapp.MyMainProcess bw.generalactivities.log'
run 'system:linkActivity -c MyApp myapp.MyMainProcess Timer Log -s'   # -c comes FIRST

# 3. Validate — empty output means clean.
run 'system:validate MyApp'

# 4. Export.
run 'system:export -ear MyApp -path /tmp/out'
```

When unsure of a command's exact arguments, run `help <bareName>` against a scratch workspace, or consult [reference.md](reference.md).

## Verified gotchas (6.12.0 HF3) — beyond the launch mechanics

Each of these cost real debugging time; `reference.md` has the detail under each command.

| Symptom | Reality |
|---|---|
| `system:listActivityTypes` prints nothing | **Broken in this build.** Harvest type IDs with `find <REFERENCE_WS> -name '*.bwp' -exec grep -ohE 'activityTypeID="[^"]*"' {} + \| sort -u`, or use the MCP `ListActivity` tool. |
| Module not found as `MyApp.module` | `createBWApplicationModule` names the module project **`MyApp`**; the process package is the **lowercased** app name. |
| `system:linkActivity` rejects your arguments | `-c`/`-u` goes **first**, before the module name. Link is auto-named `<Src>To<Dest>`. |
| Mapping `$Activity/field` produces a literal string | `configureActivity -nv` writes **literals only**; `-n` returns `Insufficient arguments for Node Mapping!` at every arity. **The CLI cannot wire activity-to-activity mappings** — use the MCP server's `configureActivityWithNodeValue`. |
| `Activity 'Mapper' does not support an output editor element` | Use `system:configureInputVariable` for a Mapper, not `configureActivityEditorSchema`. |
| `trying to replace attribute on null element:...ActivityExtensionImpl` | Benign noise from `createActivity`. Verify the model, not this line. |
| A brand-new process fails validation | Every process needs a starter (`TIBCO-BW-VALIDATION-500113`) — add a Timer, a Receive/Pick with `createInstance=true`, or a service binding. |

## When the workspace is locked

If Studio has the workspace open, every command fails with:

```
Workspace [/path/to/ws] is currently in use by another application.
```

**Do not** work around this by editing `.bwp`/`.bwm`/`MANIFEST.MF` directly, by killing Studio, or by deleting `.metadata/.lock`. Fall back to the **BW Studio MCP server**, which drives the same model APIs from inside the running IDE. If that is unreachable, ask the user to enable it under **Window ▸ Preferences ▸ BusinessWorks AI ▸ HTTP MCP Server**.

Full procedure, tool mapping, and the never-hand-edit rule: **`AGENTS.md`** at the repository root.
