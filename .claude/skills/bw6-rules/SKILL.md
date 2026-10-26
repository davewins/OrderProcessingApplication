---
name: bw6-rules
description: TIBCO BusinessWorks 6.x quality/design rules and metrics used to review BW processes, shared resources, module properties, and project structure. Use when the user asks to review, audit, lint, or check a BW6 project for coding conventions, security posture (TLS, hardcoded credentials, policy on bindings), transaction/checkpoint placement, JDBC/JMS/HTTP misuse, XML/XSD namespace hygiene, activity/process metrics, or SonarQube-style rule violations. Also use when the user references a specific rule name (e.g. "JMSHardCoded", "CheckpointProcessJDBC", "EndpointURIFromHTTPBindingSetUsingProperty") or asks "is this compliant with BW6 best practices".
---

# BW6 Quality Rules & Metrics

A curated set of **62 quality rules** and **25 metrics** for TIBCO BusinessWorks 6.x. Use them to review BW projects for correctness, security, performance, and conformance to established design conventions.

## Entry points

- **[RULES.md](RULES.md)** — index of all 62 rules with type (Project / Process / Resource), whether the rule takes parameters, its default enabled state, and a one-line description. Start here when browsing.
- **[METRICS.md](METRICS.md)** — the 25 BW6-specific metrics (data formats, HTTP/JMS/JDBC connection counts, activity counts, etc.) usable in SonarQube quality gates.

Each rule has its own detail file in this folder — click through from `RULES.md`, or open the rule's `.md` file directly by name (e.g. `JMSHardCoded.md`, `CheckpointProcessJDBC.md`).

## Rule categories

Rules cluster into these themes — useful when scoping a review:

| Theme | Representative rules |
|---|---|
| **Security / transport** | `SSLServerConnectorShouldHaveTLSprotocol`, `SSLClientConnectorShouldHaveTLSprotocol`, `HttpConnectorShouldHaveConfidentiality`, `HttpClientSSLShouldHaveConfidentiality`, `JMSConnectorShouldHaveConfidentiality`, `JKSValidation`, `OnlyOneKeystoreApplicationModule` |
| **Auth / policy on bindings** | `BindingShouldHavePolicyAssociated`, `BindingShouldNotHaveHTTPBasicPolicyAssociated`, `HttpClientMustBeUsedinHTTPBinding` |
| **Config hygiene** | `JMSHardCoded`, `JDBCHardCoded`, `PomXmlVersionsHarcoded`, `BwSharedResourceUsingModuleProperty`, `EndpointURIFromHTTPBindingSetUsingProperty`, `SharedResourcesNotUsed`, `NumberOfPropertiesSameGroup` |
| **Checkpoint / transaction placement** | `CheckpointProcessHTTP`, `CheckpointProcessREST`, `CheckpointProcessJDBC`, `CheckpointProcessTransaction`, `JDBCTransactionParallelFlow`, `CriticalSection`, `DeadlockDetection` |
| **JMS correctness** | `JMSAcknowledgementMode`, `JMSReceiverPlusConfirm`, `JMSRequestReplyNonPersistent` |
| **JDBC correctness** | `JDBCWildcards`, `ThreadpoolUsageInJDBCActivities` |
| **XML / XSD / SOAP** | `DefaultTargetNamespace`, `XMLResourceSameTargetNamespace`, `ParseXMLBinary`, `ParseXMLFromRender`, `ParseXMLRenderXMLActivity`, `RenderXMLBinary`, `RenderXmlPrettyPrint`, `SwaggerValidation`, `XPathCheck` |
| **Process structure** | `AtLeastOneStarter`, `ChoiceWithNoOtherwise`, `NoOtherwiseCheck`, `OnlyOneOtherwiseCheck`, `MultipleTransitions`, `TransitionLabels`, `UnneededEmptyActivity`, `UnneededGroup`, `LastActivityAndEndActivity`, `SubProcessInlineCheck`, `LogSubprocess`, `ForEachMapping` |
| **Process conventions** | `ProcessNamingConvention`, `ProcessNoDescription`, `ProcessWithoutTest`, `ExceptionHandlingCheck` |
| **Project structure** | `ProjectStructure`, `IsMavenProject`, `BWVersionCheck`, `NumberOfActivities`, `NumberOfExposedServices` |
| **File / binary handling** | `ListFileActivityToCheckFileExistence`, `SFTPPutBinary`, `GetFragmentBinary` |

## How to use during a review

1. Start from `RULES.md` and pick the rules relevant to the artifact under review (a REST-only process rarely needs the JMS or SFTP rules).
2. For each finding, cite the rule file (e.g. `bw6-rules/JMSHardCoded.md`) so the developer can read the full rationale.
3. Combine with the [[bw6design]] skill when the fix requires re-authoring the BW project via `bwdesign` CLI or the Business Studio MCP server.
4. Combine with any [[pattern-rest-to-ems]] / [[pattern-rest-to-db-query]] / etc. skill under `patterns/` when the review is anchored to a specific integration pattern — those skill files already reference the most relevant rules per pattern.

## Coverage notes

- **62 rules + 25 metrics** as of this bundle. `RULES.md` is authoritative if you need the exact current list.
- Some rules are **disabled by default** or **template rules** requiring parameter instantiation — check the rule file for `Initial State` and `Has Parameters` before assuming it fires out of the box.
- Rule severity is not encoded here — pair with the target SonarQube quality gate to decide which findings block a release vs. which are advisory.
