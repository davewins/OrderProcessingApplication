# CLAUDE.md

All guidance for working with TIBCO BusinessWorks 6 projects in this repository lives in **[AGENTS.md](AGENTS.md)**.

**Read [AGENTS.md](AGENTS.md) before creating or modifying any BW project.** It is the authority on which tool to use and how.

Two rules are load-bearing enough to repeat here:

1. **Never directly edit BW source files** — `.bwp`, `module.bwm`, `MANIFEST.MF`, `.substvar`, `.jsv`/`.msv`, WSDLs, XSDs, shared resources, `.bwt`. Author them *only* through the `bwdesign` CLI or the BW Studio MCP server. See the full rule and its rationale in [AGENTS.md](AGENTS.md#-the-hard-rule-never-directly-edit-bw-source-files).

2. **`bwdesign` CLI is the default.** Fall back to the Studio MCP server only when the CLI reports `Workspace [...] is currently in use by another application`. If the MCP server is unreachable, stop and ask the user to enable it in **Window ▸ Preferences ▸ BusinessWorks AI ▸ HTTP MCP Server** — do not work around it. See [AGENTS.md](AGENTS.md#which-interface-to-use--decision-procedure).
