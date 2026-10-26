# bwdesign — Full Command Reference (BW 6.12.0 HF3)

Captured from `help <command>` for every command. Syntax tokens in `[]` are optional, `<>` required, `|` alternatives. For `help`, use the **bare** name (e.g. `help createProcess`), not the `area:command` form.

> Note: several commands' help text reuses a generic boilerplate "ARGUMENTS" block (e.g. the "Name of Test Folder will be 'Tests'" line). Trust the **SYNTAX** line and the command-specific argument names; the boilerplate is copy-paste noise from the tool.

---

## Shell & workspace

### cd
Change the current working directory.
`cd <path>`

### pwd
Print the current working directory.

### ls
List projects in the workspace, or files on disk.
`ls [-f|-p] [-a]`
- `-p` list projects in the current workspace
- `-f` list files in the file system
- `-a` include hidden entities

### clear
Clear the command-line console.

### clean
Clean the comma-separated projects in the workspace. (Also available as the launch option `-clean`.)

### edition
Print the edition of this BW Studio.

### setedition
Set the edition of all workspace projects to this Studio's edition (and convert them), unless options narrow the scope.
- `-f` run without confirmation prompt
- `-name <project[,project]*>` only these projects
- `-t <bwe,bwcf,bwcloud>` target deployment edition(s); default `bwe`

### execute
Run a batch script file containing a set of commands executed in sequence.
`execute <file>`

### exit
Exit the command-line tool.

---

## Projects, import/export & build

### system:create
Create resource(s) in the workspace.
`system:create [options]`
- `application [name] [modules] -v [version]` create an application project including the given module(s); version format `major.minor.micro.qualifier` (e.g. `1.0.0.qualifier`)
- `-f` create even if it already exists
- `-verbose` print workspace info while creating
- `--help`

### system:createBWApplicationModule
Create a BW Application project (with application module + main process) in the current workspace.
`system:createBWApplicationModule [BW application name] [main process name] [version]`

> **Verified naming (6.12.0 HF3).** `system:createBWApplicationModule HelloWorld Demo 1.0.0` produces:
> - module project `HelloWorld/` — **not** `HelloWorld.module`; pass plain `HelloWorld` as the Module Name to every later command
> - application project `HelloWorld.application/`
> - the main process at `Processes/helloworld/Demo.bwp`, i.e. package = **lowercased application name**, so its qualified name is `helloworld.Demo`
>
> Getting either of these wrong is the most common cause of "activity/process not found" errors downstream. Confirm with `ls -R` or `system:display` before continuing.

### system:delete
Delete resource(s) from the workspace.
`system:delete [options]`
- `[projects]` comma-separated list of projects to delete
- `-f` force delete
- `--help`

### system:import
Import flat or zip projects into the current workspace.
`system:import [options] files`
- `files` comma-separated folders containing flat projects to import (zips ignored by default)
- `-z, -zip` the arguments are zip archive files (comma-separated)
- `-fz, -fzip` the arguments are folders containing zip archives to import
- Output per project: `imported` | `ignored` | `failed {message}`

### system:export
Generate BW artifacts (zip or EAR) into a folder from workspace projects.
`system:export [options] [projects] -path`
- `projects` comma-separated; applications export as EAR
- `[outputfolder]` / `-path` destination (defaults to local folder)
- `-e, -ear` export application as a deployable EAR (default; apps only)
- `-z, -zip` export model as zip (not with `-ear`)
- `-bin, -binary` export shared model as binary shared module (with `-zip`, not `-ear`)
- `-name [name]` name for the exported module
- `-noprofile` export without any profile
- `-pf, -profile name` export the named profile (NOTE: marked not yet implemented)
- `-pr, -property` export the module properties file to the destination
- `-substvar` export the substvar file of a given profile
- `-alsomoduleproperty` include only application properties in property file
- `-includesystem` include system properties in property file
- `-removeunused` exclude unused resources from the EAR
- `-removediagraminfo` exclude diagram info from process files in the EAR
- `-force` export even with validation errors
- `-t` tokenize the properties file for Container-deployment-target projects
- `-dot` use `.` as a separator
- `-cxf` install the Custom XPath Function project given as argument
- `--help`

### system:importpreferences
Import preferences into the current workspace.
`system:importpreferences [options] <file>` — `file` = absolute path to preferences file. Output: `imported` | `ignored` | `failed {message}`.

### system:validate
Validate BW modules in the workspace.
`system:validate [options] [modules]`
- `modules` comma-separated; defaults to all modules in the workspace
- `-d, --directory <path>` directory to store the validation result
- `-h, --help`

> **Verified (6.12.0 HF3).** **Empty output means the module is clean** — there is no "success" message. Errors print one per line as `[ERROR]TIBCO-BW-VALIDATION-<code>:...` with a `Resolution-` hint.
>
> Errors you will hit on a freshly authored process:
>
> | Code | Meaning | Fix |
> |---|---|---|
> | `500113` | process has no starter or service | add a Timer / Receive with `createInstance=true` / service binding |
> | `500103` | activity has no input transition | `system:linkActivity` from an upstream activity |
> | `500021` / `500023` | activity input/output not configured | `system:configureInputVariable` |
> | `500412` | process variable has no type | same — typing the variable fixes it |
> | `DT-VALIDATION-500006` | required input node has no value | `system:configureActivity -nv` |

### generatepom (generatepom:generatepom)
Generate `pom.xml` for the specified project.
`generatepom <Project Name>` — project = BW Application, Shared Module, or CXF Module.

### addDependency (addDependency:addDependency)
Add a dependency to an Application Module's `pom.xml`.
`addDependency [Application Module] [Group ID] [Artifact ID] [Version]`

### copyArtifacts
Copy/paste a BW file with updated references.
`copyArtifacts <option> <FROM_FILE> <TO_FOLDER>`
- optional new resource name (without extension) to rename the copy
- `-f` overwrite if already present at destination
- `-s` skip dependency copy

### diagram:gen_diagrams
Save each process diagram of a project as SVG.
`diagram:gen_diagrams [project]` — optional output folder argument to choose where diagrams are saved.

### generate_manifest_json
Create `manifest.json` (or EAR) from a BW EAR file.
`generate_manifest_json [options] [ear_location] [manifest_location]`
- `-project <name>` project to generate the Manifest.json for
- `-ear` generate a new EAR from an EAR built by an earlier Studio version

---

## Processes, activities, groups, links

### system:createProcess
Create an empty process or subprocess in an imported project.
`system:createProcess [option] [moduleName] [processName] [packageName] [modifier]`
- `option`: `-p` main process, `-s` subprocess
- `modifier` (optional): `public` (default) or `private`
- `package` may be an existing or new package name

### system:createActivity
Create an empty activity in a process.
`system:createActivity [Module Name] [Process Name w/ Namespace] [Activity Type] [Parent Option] [Child Option]`
- Parent Option (optional): `-g` group, `-c` catch, `-ca` catchall
- Child Option (optional): for catch = index (e.g. `0`); for groups: `-s` Scope, `-r` Repeat, `-w` While, `-f` ForEach, `-i` Iterate, `-re` RepeatOnError, `-c` Critical Section, `-l` Local Transaction
- get valid Activity Types from `system:listActivityTypes` — **but see the warning under that command; it returns nothing in this build**

> **Verified (6.12.0 HF3).**
> - The activity is created with a **default name derived from its type** (`bw.generalactivities.log` → `Log`, then `Log1`, `Log2`, … for repeats). The command does **not** print the name. Read it back from the `.bwp` or rename with `system:renameActivity`.
> - `createActivity` emits `trying to replace attribute on null element:class com.tibco.bx.bpelExtension.extensions.impl.ActivityExtensionImpl` on stderr. This is **benign noise** — the activity is created correctly. Verify the model, not this line.
>
> Working example:
> ```bash
> printf 'system:createActivity HelloWorld helloworld.Demo bw.generalactivities.mapper\nexit\n' \
>   | ./bwdesign -data "$WS" -silent 2>&1 | sed 's/\x1b\[[0-9;]*m//g'
> ```

### system:renameActivity
`system:renameActivity [project Name] [Process Name w/ Namespace] [Activity Name] [New Activity Name]`

### system:linkActivity
Create or update a link between activities.
`system:linkActivity [Project] [Process w/ Namespace] {[Src Activity] [Dest Activity] | [Link Name]} [Link Type] [Expression]`
- Operation: `-c` create (needs Src + Dest), `-u` update (needs Link Name)
- Link Type: `-s` success, `-sc` success-with-condition, `-e` error, `-o` otherwise, `-se` success-with-condition needing an XPath expression
- Expression required only when link type is `-se`

> **Verified argument order (6.12.0 HF3).** The SYNTAX line above omits where the operation option goes — it comes **first**, before the module name:
> ```bash
> system:linkActivity -c HelloWorld helloworld.Demo Mapper Log -s
> #                   ^^ operation      module     process    src   dest  ^^ link type
> ```
> The link is auto-named `<Src>To<Dest>` (e.g. `MapperToLog`); that generated name is what `-u` expects later. Success is silent — confirm by grepping `<bpws:links>` in the `.bwp`.

### system:configureActivity
Configure a particular activity.
`system:configureActivity [option] [Module] [Process w/ Namespace] [Activity type] [attribute/node] [value]`
- `-v` configure an attribute with a value
- `-n` configure a source node
- `-nv` configure a particular node with a value
- `-e` bind to a process property or module property

The `[Activity type]` argument is really the **activity name** (e.g. `Log`), not the type ID.

> **⚠ Verified limits (6.12.0 HF3) — these two shape how you design a process.**
>
> **`-nv` writes literals only.** It wraps the value in XPath string quotes, so it can never reference another activity:
> ```bash
> system:configureActivity -nv HelloWorld helloworld.Demo Mapper message "Hello World"
> # →  <xsl:value-of select="&quot;Hello World&quot;"/>
> ```
> Passing `$Mapper/message` as the value yields the *string* `"$Mapper/message"`, not a mapping.
>
> **`-n` (source-node mapping) could not be made to work.** Every arity tried returns `[ERROR]Insufficient arguments for Node Mapping!`:
> ```
> system:configureActivity -n <Mod> <Proc> Log message $Mapper/message
> system:configureActivity -n <Mod> <Proc> Log message Mapper message
> ```
>
> **Consequence:** the CLI alone cannot wire one activity's output into another's input. When a spec needs a real activity-to-activity mapping, use the **Studio MCP server** — `configureActivityWithNodeValue` accepts `$Activity/field` — rather than hand-editing the `.bwp`. See `AGENTS.md` at the repo root.

### system:configureInputVariable
Configure the input variable (schema element) for an activity.
`system:configureInputVariable [Module] [Process w/ Namespace] [Activity type] [schemaFileName] [schemaElement]`

> **Verified (6.12.0 HF3).** This is **how you give a Mapper its schema** — `system:configureActivityEditorSchema` rejects the Mapper activity outright. One call types *both* the input and output variables:
> ```bash
> system:configureInputVariable HelloWorld helloworld.Demo Mapper Greeting.xsd Greeting
> # → <bpws:variable element="ns2:Greeting" name="Mapper-input" .../>
> # → <bpws:variable element="ns2:Greeting" name="Mapper"       .../>
> ```
> Prints `Input Variable Configured Successfully`. The XSD must already exist in the module's `Schemas/` folder and the element must be a **global** element.
>
> **Schema design tip:** prefer `elementFormDefault="unqualified"` in XSDs you author for BW. With `qualified`, every generated XPath needs a namespace prefix that the tooling does not always declare, which turns into `No matching <node>` validation errors that are tedious to fix.

### system:configureActivityEditorSchema
Configure the Input or Output Editor schema of an activity (e.g. Render JSON / Parse JSON) using a global XSD element from the project's `Schemas` folder.
`system:configureActivityEditorSchema <Module> <Process> <Activity> <input|output> <XSD File Name> <Element Name>`

> **Verified (6.12.0 HF3).** Only works on activities that actually have an Input/Output **Editor** tab. On a Mapper it fails with:
> ```
> [ERROR]Activity 'Mapper' does not support an output editor element.
>        Only activities with an Input/Output Editor tab (e.g. Render JSON, Parse JSON) support this.
> ```
> Use `system:configureInputVariable` for a Mapper.

### system:createGroup
Create a group around/within a process.
`system:createGroup [Options] [project] [package Namespace] [Process Name]`
- `-r` Repeat, `-re` RepeatOnError, `-s` Scope, `-c` Critical Section, `-f` ForEach, `-w` While, `-i` Iterate, `-l` Local Transaction

### system:createCatch
Create an empty catch area in a process.
`system:createCatch [Module] [Process w/ Namespace] [Catch Type]`
- `-catch` empty catch block (any number)
- `-catchall` exception catch-all block (only once)

### system:setCallProcessName
Set the called subprocess on a Call Process activity.
`system:setCallProcessName [project] [Source Process w/ Namespace] [Call process name] [Sub Process w/ Namespace]`

### system:listActivityTypes
List all valid Activity Types with their Name and ID. (No arguments.)

> **⚠ BROKEN in 6.12.0 HF3 — returns no output at all.** Confirmed empty both piped on stdin and passed as a direct argument, against both a scratch workspace and a workspace containing projects. `help listActivityTypes` works and describes it, but the command itself prints nothing.
>
> **Workaround — harvest the IDs from any workspace of existing BW projects.** Recurse; processes nest below the package folder, so a fixed-depth glob misses most of them:
> ```bash
> find <REFERENCE_WS> -name '*.bwp' \
>   -exec grep -ohE 'activityTypeID="[^"]*"' {} + | sort -u
> ```
> IDs verified this way against a workspace of BW 6.12 sample projects:
>
> | Activity | Type ID |
> |---|---|
> | Mapper | `bw.generalactivities.mapper` |
> | Log | `bw.generalactivities.log` |
> | Timer (process starter) | `bw.generalactivities.timer` |
> | Sleep | `bw.generalactivities.sleep` |
> | Assign | `bw.generalactivities.bwassign` |
> | Read File | `bw.file.read` |
> | Write File | `bw.file.write` |
> | JDBC Query | `bw.jdbc.JDBCQuery` |
> | JDBC Update | `bw.jdbc.update` |
> | Parse JSON | `bw.restjson.JsonParser` |
> | Render JSON | `bw.restjson.JsonRender` |
> | REST (invoke) | `bw.restjson.Rest` |
>
> Call Process is `bw.generalactivities.callprocess` (referenced by `system:setCallProcessName`, not present in the sample `.bwp` files above).
>
> If the Studio MCP server is reachable, its `ListActivity` / `ListActivityWithIcons` tools return the full palette and are the better source.

### system:addtag
Add one or more tag elements to an activity.
`system:addtag <moduleName> <processName> <activityName> <tagName>[,<tagName>...]`

### system:configuretag
Configure an existing tag on an activity (rename and/or set XPath).
`system:configuretag <moduleName> <processName> <activityName> <tagName> [-n <newName>] [-e <expression>]`

### system:bwPackage
Create/update/delete a package in a module.
`system:bwPackage [options] [Module Name] [package Namespace]`
- `-c` create a package
- `-d` delete a package
- `-u [Module] [Old Namespace] [New Namespace]` rename a package

### system:addjdbcparam
Add a parameter to a JDBC activity (Query, Update, or Call Procedure). Use standard SQL type names.
`system:addjdbcparam <moduleName> <processName> <activityName> <paramName> <dataType> [-d IN|OUT|INOUT]`
- `dataType` e.g. `VARCHAR`, `INTEGER`, `DATE`, `TIMESTAMP`, `NUMERIC`, `CLOB`, `BLOB`
- `-d` direction for Call Procedure (default `IN`)

---

## Services, WSDL, SOAP/REST bindings, Swagger

### system:createWSDL
Create a WSDL for a process.
`system:createWSDL [Module] [Process w/ Namespace] [WSDL Name] [Operation Type]`
- Operation type: `-ImplementOperation`, `-ImplementConstructorOperation`, `-ImplementProxyOperation`, `-InvokeOperation`

### system:setServicePortType
Set the process service port type to the specified WSDL.
`system:setServicePortType [module] [process w/ Namespace] [Service Name] [WSDL File name]...`

### system:createBinding
Create a SOAP or REST binding on the process's service.
`system:createBinding [option] [module] [process w/ Namespace] [Service Name]...`
- `-s` SOAP binding, `-r` REST binding

### system:setSOAPBindingTransportType
Set the transport of a SOAP binding and attach the connector resource.
`system:setSOAPBindingTransportType [module] [process w/ Namespace] [Service Name] [Transport name] [package.connectorName]...`
- Transport: `http` or `jms`
- Connector resource name includes its package (e.g. `package.HttpConnectorResource`)

### createSwaggerOperation
Create a Swagger-based REST service or reference operation in a process.
`createSwaggerOperation [Create Option] [Operation option] [Process Name] [Module Name] [Swagger File Name] [Resource Path]`
- Create option: `-s` REST Service binding, `-r` REST Reference binding
- `-all` (service only) create a REST service with all operations at the resource path
- Operation options (reference binding): `-get`, `-put`, `-post`, `-patch`, `-delete`, `-head`, `-option`

### system:createConversation
Create a BPEL conversation (correlation set) on an activity's Conversation tab.
`system:createConversation <Module> <Process> <Activity> [<Conversation Name>]` — name auto-generated (e.g. `Conversation1`) if omitted.

### system:configureConversationKey
Set the correlation-key XPath on an existing conversation.
`system:configureConversationKey <Module> <Process> <Activity> <Conversation Name> <XPath>`

---

## Properties, variables, resources & profiles

### system:moduleProperty
Create/update/delete a module property in a module or group.
`system:moduleProperty [op option] [option] [module property path] [type] [count]`
- Path: `projectName` or `projectName/GroupName`
- Types: `string, boolean, dateTime, long, int, password, proxy, rv, smtp, sslclient, sslserver, subject, tcpconnection, threadpool, trust, oauth, notify, ldap, keystore, jms, jdbc, javaglobal, useridentity, httpconnector, httpclient, ftpconnection, dataformat`
- `-c [path] [type] [count]` create N properties
- `-u -n [path] [newName]` rename
- `-u -v [path] [newValue]` set value
- `-u -t [path] [newType]` change type
- `-d [path]` delete

### system:processProperty
Create/update/delete a process property.
`system:processProperty [options] [module] [process w/ Namespace] [PropertyName] [type] [Value]`
- `-c` create, `-d` delete, `-u -n` rename, `-u -t` change type, `-u -v` set value

### system:processVariable
Create/update/delete a process variable.
`system:processVariable [options] [module] [process w/ Namespace] [VariableName] [Value...]`
- `-c` create, `-d` delete, `-u -n` rename, `-u -t` change type, `-u -v` set value

### system:sharedVariable
Create/update/delete a Job or Module shared variable.
`system:sharedVariable [variable option] [operation option] [project] [variable type]`
- Variable option: `-j` job shared variable, `-m` module shared variable
- Operation: `-c` create, `-u` update, `-d` delete
- Type: `string, integer, boolean, dateTime`
- Update sub-options: `-u -n [proj] [name] [newName]`, `-u -t [...] [newType]`, `-u -v [...] [newValue]`

### system:display
List all module properties in a module.
`system:display [option] [module name]` — option currently supports `moduleProperty`.

### system:copyModuleProp
Copy module properties and groups between modules.
`system:copyModuleProp [options] [source module] [target module]`
- `-m [source]/Group/moduleProperty [target]` copy a specific property
- `-g [source]/Group [target]` copy specific group(s)
- `-a [source] [target]` copy all properties and groups
- `-o` override target value with source value
- `-s` skip properties already present at destination
- (default, no `-o`/`-s`) copies with a modified unique name

### system:connectResource
Assign a shared resource to a process property.
`system:connectResource [process path] [process property] [resource path]`

### createHttpConnector (createHttpConnector:createHttpConnector)
Create an HTTP Connector shared resource.
`createHttpConnector <BW Module> <Package> <HTTP connector resource name>`

### createJMSConnection (createJMSConnection:createJMSConnection)
Create a JMS Connection shared resource.
`createJMSConnection <BW Module> <Package> <JMS connection resource name>`

### createJNDIConnection (createJNDIConnection:createJNDIConnection)
Create a JNDI Connection shared resource.
`createJNDIConnection <BW Module> <Package> <JNDI connection resource name>`

### system:createApplicationProfile
`system:createApplicationProfile [applicationName] [profileName]`

### system:renameApplicationProfile
`system:renameApplicationProfile [moduleName] [oldProfileName] [newProfileName]`

### system:deleteApplicationProfile
`system:deleteApplicationProfile [moduleName] [profileName]`

### export_to_consul
Export the properties from a profile to a Consul KV store.
`export_to_consul [options]`
- `-profile <name>` (required)
- `-project <name>` BW application project containing the profile (required)
- `-consul <url>` e.g. `http://127.0.0.1:8500` (required)
- `-consultoken <token>` (optional)
- `-customkey <key>` custom encryption key for Password-type properties (optional)

---

## Testing

### createTestFolder (createTestFolder:createTestFolder)
Create the `Tests` folder for a project.
`createTestFolder [Project Name]`

### createTestSuite (createTestSuite:createTestSuite)
Create a test suite for a process under `Tests`.
`createTestSuite [Process Path] [Folder Path] [Test Suite Name]`
- Test suite name must be a valid NCName ending in `.bwts`; the folder must already exist.

### createTestFile (createTestFile:createTestFile)
Create a test file (.bwt) for one or more processes.
`createTestFile <-process|-package|-project> <BW Module> [Package] <Test File Name> <selector>`
- `-process "pkg.proc1,pkg.proc2,..."`
- `-package <packageName>` all processes in the package
- `-project <projectName>` all processes in the project
- If no package given, the test file is created in the `Tests` folder.

### addAssertion (addAssertion:addAssertion)
Add an assertion (shares the createTestFile selector/syntax).
`addAssertion <-process|-package|-project> <BW Module> [Package] <Test File Name> <selector>`
- `-input` add assertion to input
- `-output` add assertion to output
- `assertionType`: `activity` | `primitive`
- `primitiveType`: `byte|short|int|long|float|double|char|boolean`

### deleteAssertion (deleteAssertion:deleteAssertion)
`deleteAssertion [Module] [Test File Name] [Process w/ Namespace] [Activity name]`

### listAllTests (listAllTests:listAllTests)
List assertions for a process or module.
`listAllTests [moduleName] [processName]` — module required; if process omitted, lists for all processes in the module.

### mockOutputFile (mockOutputFile:mockOutputFile)
Add a mock output file to an activity.
`mockOutputFile [Module] [Test File Name (.bwt)] [Process w/ Namespace] [Activity Name] [Mock Output File Path]`
- Mock output file must exist and be XML; test file may be `Test1.bwt` or `packagename/Test1.bwt`.

---

## MCP (AI / Model Context Protocol)

### generate_mcp_skeleton
Create an MCP-ified BW skeleton project from an API spec.
`generate_mcp_skeleton [options] [api_spec] [application_module_name]`
- `api_spec` path to API spec (JSON currently supported)
- `-tool` expose new operations as tools, **or** `-resource` expose as resources (exactly one is mandatory; cannot combine)

### generate_mcp_config
Configure an existing application module project with MCP.
`generate_mcp_config [options] [application_module_name]` (module must be in the workspace)
- `-tool` expose all bindings as tools, **or** `-resource` as resources (exactly one mandatory)

### generate_mcp_ear
Generate an MCP-compatible EAR from an existing EAR.
`generate_mcp_ear [options] [ear_location] [destination_location]`
- `-tool` expose bindings as tools (optional), `-resource` as resources (optional)
- `-restport <port>` REST MCP endpoint port, **or** `-soapport <port>` SOAP MCP endpoint port (exactly one mandatory)

### system:setHttpMcpServerEnabled
Enable/disable the HTTP MCP Server (BusinessWorks AI preferences) and set host/port.
`system:setHttpMcpServerEnabled <true|false> [hostname] [port]`
- hostname defaults to `localhost`; port defaults to `8080` (1–65535)
