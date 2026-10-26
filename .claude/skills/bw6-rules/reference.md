# bw6-rules — reference

Full rule details extracted from [TIBCOSoftware/sonar-bw docs/rules/bw6](https://github.com/TIBCOSoftware/sonar-bw/tree/main/docs/rules/bw6). Each entry: what it detects, why it matters, how to fix. Grouped by scope, alphabetical within.

## Project scope (14)

### AtLeastOneStarter
- **Detects:** Application module lacks any starter process besides the Activator.
- **Why:** Every application needs at least one starter to communicate with the outside world.
- **Fix:** Create a process that includes at least one Starter activity.

### BindingShouldHavePolicyAssociated
- **Detects:** Input binding has no policy associated to authenticate incoming communications.
- **Why:** Code must be secure, so authentication is mandatory to prevent unauthorized external service calls.
- **Fix:** Attach a Policy to the binding to provide an authentication mechanism.

### BindingShouldNotHaveHTTPBasicPolicyAssociated
- **Detects:** Binding uses HTTP Basic Authentication policy, which is no longer considered secure.
- **Why:** HTTP Basic Authentication sends credentials without encryption and is no longer regarded as secure.
- **Fix:** Replace Basic Authentication with a more secure authentication method.

### BWVersionCheck
- **Detects:** BusinessWorks module version does not match the recommended, more secure and performant release.
- **Why:** Older versions accumulate vulnerabilities; keeping software current improves security and performance.
- **Fix:** Upgrade the TIBCO BusinessWorks version to the latest recommended release.

### EndpointURIFromHTTPBindingSetUsingProperty
- **Detects:** SOAP/HTTP binding endpoint URI is not driven from a Module Property.
- **Why:** Endpoint URIs vary by environment and must stay configurable without code changes.
- **Fix:** Bind the binding's Endpoint URI to a Module Property so it can be updated at deployment.

### IsMavenProject
- **Detects:** BusinessWorks module is not configured as a Maven project.
- **Why:** Maven nature enables integration with third-party tools and CI/CD pipelines.
- **Fix:** Generate the POM for the application via the BusinessWorks Maven plugin to add Maven nature.

### JKSValidation
- **Detects:** JKS keystore contains expired or self-signed certificates.
- **Why:** Secure code requires secure components; expired or self-signed certificates undermine security guarantees.
- **Fix:** Include only unexpired, CA-signed certificates in the project JKS.

### NumberOfPropertiesSameGroup
- **Detects:** A module property group contains more properties than the configured maximum.
- **Why:** Well-grouped module properties improve readability and understandability of the whole project.
- **Fix:** Break large property groups into smaller functional groups.

### OnlyOneKeystoreApplicationModule
- **Detects:** Application contains more than one Keystore shared resource.
- **Why:** Multiple keystores complicate certificate and key management within the same codebase.
- **Fix:** Consolidate certificates and keys into a single Keystore Provider resource.

### PomXmlVersionsHarcoded
- **Detects:** Dependency versions in `pom.xml` are hardcoded rather than driven by properties.
- **Why:** Hardcoded versions block clean software-currency management in Maven projects.
- **Fix:** Define `pom.xml` properties and reference them in each dependency's `<version>` tag.

### ProjectStructure
- **Detects:** Application module folder structure or naming does not match the corporate policy.
- **Why:** A consistent structure supports best practices and collaboration across the organization.
- **Fix:** Adjust folder layout and naming to comply with the corporate project structure policy.

### SwaggerValidation
- **Detects:** Swagger API definition file in the project is invalid or violates best practices.
- **Why:** Valid Swagger ensures cross-tool compatibility and adherence to the interface standard.
- **Fix:** Resolve Swagger validation errors so the definition complies with the standard.

### XMLResourceSameTargetNamespace
- **Detects:** Multiple XML Schema or WSDL files share the same target namespace.
- **Why:** Namespaces scope XML definitions; reusing the same namespace across different resources is a bad practice.
- **Fix:** Give each XML resource its own unique target namespace.

### XPathCheck
- **Detects:** Template rule for user-defined custom XPath checks against the application.
- **Why:** Custom XPath checks let teams enforce use-case-specific quality checks not covered by built-in rules.
- **Fix:** Configure the template with the desired XPath expression; remediation depends on the check.

## Process scope (41)

### CheckpointProcessHTTP
- **Detects:** Checkpoint activity is placed right after or in parallel with HTTP activities.
- **Why:** A recovered instance cannot reply on a closed HTTP socket, so the checkpoint's promise of resumability is false for HTTP starters.
- **Fix:** Redesign for a stateless approach so no Checkpoint is required near HTTP activities.

### CheckpointProcessJDBC
- **Detects:** Checkpoint is placed after or in parallel with JDBC Query or other idempotent activities.
- **Why:** Idempotent activities can safely re-run without a checkpoint; checkpointing them just wastes storage and time.
- **Fix:** Place checkpoints only after non-idempotent JDBC writes, not after queries or idempotent flows.

### CheckpointProcessREST
- **Detects:** Checkpoint activity is placed near REST/HTTP activities where recovery cannot respond to the caller.
- **Why:** Same as CheckpointProcessHTTP — the HTTP/REST socket is gone after recovery, so the reply cannot be sent.
- **Fix:** Design processes to be stateless so Checkpoints are not needed near REST activities.

### CheckpointProcessTransaction
- **Detects:** Checkpoint is placed inside or in parallel with a Transaction or Critical Section group.
- **Why:** Checkpoints inside these groups do not survive rollback and can leave data in an inconsistent state.
- **Fix:** Move Checkpoints outside transaction and critical section groups, at reliably reachable points.

### ChoiceWithNoOtherwise
- **Detects:** Choice statement in activity input mapping is missing an otherwise branch.
- **Why:** A missing otherwise leaves non-matching cases with undefined mapping behaviour.
- **Fix:** Add an otherwise option to the mapper's choice element with a relevant mapping.

### CriticalSection
- **Detects:** Critical Section group contains waiting or long-running activities (Request/Reply, Wait, Sleep).
- **Why:** Long waits inside a critical section block other instances, degrading throughput and increasing resource use.
- **Fix:** Keep critical sections minimal and add realistic timeouts to any external-facing activity.

### DeadlockDetection
- **Detects:** Process design contains a potential deadlock or infinite loop between parallel flows or subprocesses.
- **Why:** Deadlocks block processes waiting on each other's resources; infinite loops waste compute and cause failures.
- **Fix:** Redesign or split processes to eliminate circular dependencies and self-recursive loops.

### DefaultTargetNamespace
- **Detects:** Process uses the default target namespace generated by the tool.
- **Why:** XML namespaces qualify names; processes should follow a deliberate namespace strategy, not tool defaults.
- **Fix:** Change the process namespace in the Advanced tab as soon as the process is created.

### ExceptionHandlingCheck
- **Detects:** Component process has no exception handling for internal errors.
- **Why:** Unhandled errors leak internal failures to callers and lead to unpredictable, insecure behavior.
- **Fix:** Add a Catch activity in the component process to handle exceptions before they reach callers.

### ForEachMapping
- **Detects:** Activity input mapping uses For-Each where Copy-Of would suffice.
- **Why:** Copy-Of is more performant and uses less memory than for-each iteration from the root.
- **Fix:** Replace For-Each with Copy-Of when possible, or annotate the activity with `SQIGNORE:ForEachMapping`.

### GetFragmentBinary
- **Detects:** GetFragment activity is not configured in binary mode.
- **Why:** Binary mode reads large XML fragments faster and stores results with less memory than text mode.
- **Fix:** Enable Binary Mode on the Get Fragment activity's configuration.

### HttpClientMustBeUsedinHTTPBinding
- **Detects:** HTTP reference binding uses a plain URL instead of an HTTP Client shared resource.
- **Why:** An HTTP Client resource controls the connection pool and lets properties expose configuration to avoid hardcoding.
- **Fix:** Attach an HTTP Client shared resource to the HTTP-based reference binding.

### JDBCHardCoded
- **Detects:** JDBC activity has hardcoded Timeout or MaxRows values.
- **Why:** Timeout and row limits depend on runtime environment and data, so they must remain configurable.
- **Fix:** Bind JDBC activity Timeout and MaxRows to Process or Module properties.

### JDBCTransactionParallelFlow
- **Detects:** JDBC Transaction Group contains parallel flows with JDBC activities.
- **Why:** Parallel flows inside a JDBC Transaction Group are not executed, yielding unexpected transaction output.
- **Fix:** Remove the parallel design from inside JDBC Transaction Groups.

### JDBCWildcards
- **Detects:** JDBC activity query uses a wildcard (`*`) instead of listing fields explicitly.
- **Why:** `SELECT *` is fragile against schema changes and pulls unnecessary columns, hurting reliability and performance.
- **Fix:** Rewrite JDBC queries to list the specific field names needed.

### JMSAcknowledgementMode
- **Detects:** JMS receiver activity uses AUTO acknowledgement mode instead of CLIENT ACK.
- **Why:** AUTO acknowledges on receipt, so failures during processing lose the message with no server-side retry.
- **Fix:** Use CLIENT ACK mode and add a Confirm activity after successful processing.

### JMSHardCoded
- **Detects:** JMS activity has hardcoded Timeout, Destination, Reply Destination, Message Selector, or Polling Interval.
- **Why:** Hardcoding EMS-facing values is bad practice because they change with environment and workload.
- **Fix:** Bind JMS activity Timeout, Destination, Reply Destination, Message Selector, and Polling Interval to Properties.

### JMSReceiverPlusConfirm
- **Detects:** JMS Receiver with CLIENT ACK mode lacks a Confirm activity on every successful flow.
- **Why:** Without Confirm on OK paths, EMS retries messages already processed, causing duplicates and unexpected behavior.
- **Fix:** Add a Confirm activity at the end of every successful flow following a CLIENT ACK JMS Receiver.

### JMSRequestReplyNonPersistent
- **Detects:** JMS Request/Reply activity is configured to send PERSISTENT messages.
- **Why:** Request/Reply is synchronous, so persisting the message adds no value but costs performance.
- **Fix:** Set the JMS Request/Reply Advanced configuration delivery mode to NON_PERSISTENT.

### LastActivityAndEndActivity
- **Detects:** Process flow does not terminate with a proper end activity (End, Reply, Exit, Throw, Rethrow).
- **Why:** Explicit end activities make it clear the flow intentionally ends there and improve maintainability.
- **Fix:** Terminate every flow with an End, Reply, Exit, Throw, or Rethrow activity.

### ListFileActivityToCheckFileExistence
- **Detects:** List File activity is used only to check whether a single file exists.
- **Why:** Read File is faster and independent of folder size for existence checks compared to listing directory contents.
- **Fix:** Use Read File with fileContent unchecked instead of List File to test existence.

### LogSubprocess
- **Detects:** Log activity is used directly in a component process rather than a dedicated logging subprocess.
- **Why:** Centralizing logging in a subprocess makes auditing consistent and reusable across the project.
- **Fix:** Move logging and auditing code into a shared subprocess and invoke it from where needed.

### MultipleTransitions
- **Detects:** Multiple parallel transitions merge back into a regular activity instead of an Empty activity.
- **Why:** An Empty activity is the intended join point; using it improves readability of merged parallel flows.
- **Fix:** Insert an Empty activity where multiple transition flows merge back into a single flow.

### NoOtherwiseCheck
- **Detects:** Set of transitions from an activity has no otherwise (no-matching-condition) path.
- **Why:** A missing otherwise leaves non-happy-path cases unhandled and the design incomplete.
- **Fix:** Add an Otherwise transition covering the non-matching case with appropriate behaviour.

### NumberOfActivities
- **Detects:** Process contains more activities than the configured readability threshold.
- **Why:** Too many activities hurt process readability and maintainability.
- **Fix:** Refactor by extracting logic into subprocesses to keep activity counts manageable.

### NumberOfExposedServices
- **Detects:** Process exposes more services than the configured readability threshold.
- **Why:** Too many exposed services in one process hurt readability and maintainability.
- **Fix:** Split services across additional processes, grouped by function or one service per process.

### OnlyOneOtherwiseCheck
- **Detects:** Multiple Otherwise (no-matching-condition) transitions exist from a single activity.
- **Why:** Multiple otherwise transitions are unsupported and can cause unexpected runtime behavior.
- **Fix:** Keep one Otherwise transition; route parallel work through an Empty activity if needed.

### ParseXMLBinary
- **Detects:** ParseXML activity is configured in text mode instead of binary.
- **Why:** Binary mode is significantly faster than text mode for ParseXML operations.
- **Fix:** Switch ParseXML and its upstream activity to binary mode.

### ParseXMLFromRender
- **Detects:** ParseXML activity consumes the output of a `tib:render-xml` call.
- **Why:** Going Parsed → Text → Parsed is wasted work and hurts readability when coercion would suffice.
- **Fix:** Remove the render/parse round-trip and use coercion on the next activity that needs the data.

### ParseXMLRenderXMLActivity
- **Detects:** ParseXML activity consumes the output of a RenderXML activity.
- **Why:** Going Parsed → Text → Parsed is wasted work and hurts readability when coercion would suffice.
- **Fix:** Remove the RenderXML/ParseXML pair and use coercion on the next activity that needs the data.

### ProcessNamingConvention
- **Detects:** Process name does not match the configured naming convention pattern.
- **Why:** Naming conventions ease maintenance and understanding across a team of developers.
- **Fix:** Rename the process to match the configured naming convention pattern.

### ProcessNoDescription
- **Detects:** Process has no description populated.
- **Why:** In-process descriptions are embedded documentation that eases maintenance and readability.
- **Fix:** Populate each process description with accurate, useful information.

### ProcessWithoutTest
- **Detects:** Process has no associated unit test file.
- **Why:** Unit tests validate correct behavior and protect against regressions as the process evolves.
- **Fix:** Create at least one unit test case for each process using the built-in BusinessWorks testing capabilities.

### RenderXMLBinary
- **Detects:** RenderXML activity is configured in text mode instead of binary.
- **Why:** Binary mode is significantly faster than text mode for RenderXML operations.
- **Fix:** Switch RenderXML and its upstream activity to binary mode.

### RenderXmlPrettyPrint
- **Detects:** `tib:render-xml` call has pretty-print set to true.
- **Why:** Pretty-print adds indentation cost and breaks log-aggregation pipelines with extra newlines.
- **Fix:** Set the pretty-print option on render-xml calls to `false`.

### SFTPPutBinary
- **Detects:** SFTP Put activity is not configured for binary transmission.
- **Why:** ASCII mode makes transfers depend on file encoding and risks silent transformations.
- **Fix:** Set the SFTP Put activity's transmission option to Binary.

### SubProcessInlineCheck
- **Detects:** Large data payload is passed each time to an Inline SubProcess.
- **Why:** Passing large payloads inline is expensive; a Job Shared Variable is more performant.
- **Fix:** Move large data into a Job Shared Variable instead of passing it to the Inline SubProcess.

### ThreadpoolUsageInJDBCActivities
- **Detects:** JDBC activity does not have a ThreadPool Resource assigned.
- **Why:** JDBC activities are async and use an unbounded internal pool, risking memory growth and performance issues.
- **Fix:** Assign a ThreadPool Resource instance in the JDBC activity's Advanced tab to bound thread usage.

### TransitionLabels
- **Detects:** Success With Condition (XPath) transition has no descriptive label.
- **Why:** Labels on conditional transitions explain the intent and improve readability and maintenance.
- **Fix:** Add a descriptive label to each conditional transition.

### UnneededEmptyActivity
- **Detects:** Process contains Empty activities that serve no purpose.
- **Why:** Superfluous Empty activities clutter the diagram and hurt readability.
- **Fix:** Remove unneeded Empty activities and fold their behaviour into the next activity's input mapping.

### UnneededGroup
- **Detects:** Process contains groups that add no value beyond what input mapping could provide.
- **Why:** Unnecessary groups obscure the process; loops often belong inside an activity's input mapping.
- **Fix:** Remove unnecessary groups and express the equivalent logic in the activity's input mapping.

## Resource scope (7)

### BwSharedResourceUsingModuleProperty
- **Detects:** Shared resource parameters are hardcoded instead of using module properties.
- **Why:** Hardcoding deployment-sensitive values like external-resource endpoints is a bad practice.
- **Fix:** Bind critical shared resource parameters to Module Properties.

### HttpClientSSLShouldHaveConfidentiality
- **Detects:** HTTP Client using port 443 does not have confidentiality (SSL) settings configured.
- **Why:** SSL confidentiality is required to protect HTTP communications from eavesdropping.
- **Fix:** Enable confidentiality options on the HTTP Client resource.

### HttpConnectorShouldHaveConfidentiality
- **Detects:** HTTP Connector resource does not have confidentiality (SSL) settings configured.
- **Why:** SSL confidentiality is required to protect HTTP communications from eavesdropping.
- **Fix:** Enable confidentiality options on the HTTP Connector resource.

### JMSConnectorShouldHaveConfidentiality
- **Detects:** JMS Connector resource does not have confidentiality (SSL) settings configured.
- **Why:** SSL confidentiality is required to protect JMS communications from eavesdropping.
- **Fix:** Enable confidentiality options on the JMS Connector resource.

### SharedResourcesNotUsed
- **Detects:** Project contains shared resources that are not referenced anywhere.
- **Why:** Unused shared resources hurt readability and maintainability of the project.
- **Fix:** Delete unused shared resources from the project.

### SSLClientConnectorShouldHaveTLSprotocol
- **Detects:** SSLClient Connector does not use a recommended TLS protocol version.
- **Why:** Older SSL versions are no longer considered secure for client/server communications.
- **Fix:** Upgrade the SSLClient Connector to TLS 1.0 or higher.

### SSLServerConnectorShouldHaveTLSprotocol
- **Detects:** SSLServer Connector does not use a recommended TLS protocol version.
- **Why:** Older SSL versions are no longer considered secure for client/server communications.
- **Fix:** Upgrade the SSLServer Connector to TLS 1.0 or higher.
