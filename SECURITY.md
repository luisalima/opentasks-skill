# Security Policy

OpenTasks is a documentation convention and agent skill. It does not run a service, process private data by itself, or ship executable code today.

## Reporting a vulnerability

Please do not open a public issue with sensitive security details.

Use GitHub private vulnerability reporting if it is enabled for this repository. If it is not available, open a minimal public issue asking for a private contact path and do not include exploit details, secrets, or private repository information.

## Scope

Security-relevant issues may include:

- Instructions that could cause agents to expose secrets.
- Unsafe guidance for writing or modifying files.
- Generated conventions that encourage committing sensitive data.
- Future scripts or tooling that mishandle paths, shell commands, or untrusted input.

Out of scope:

- General feature requests.
- Disagreements about task format or naming.
- Vulnerabilities in unrelated tools used alongside OpenTasks.
