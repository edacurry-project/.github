# Security policy

## Reporting a vulnerability

Please do not disclose security-sensitive issues in a public GitHub issue.

For a vulnerability affecting a specific EDACurry repository, use that repository's private vulnerability reporting / security advisory interface when available. If private vulnerability reporting is not enabled, contact the EDACurry organization maintainers privately through GitHub before sharing exploit details publicly.

When reporting a vulnerability, include enough information to reproduce and assess the issue:

- affected repository and revision/tag;
- affected component or API;
- impact and realistic attack conditions;
- minimal reproducer or proof of concept when safe to share privately;
- operating system/toolchain if relevant;
- any known mitigation or workaround.

## Scope

Security reports may include issues in:

- C++ parsing of untrusted or malformed netlists;
- memory safety and resource exhaustion;
- unsafe file/path handling;
- generated output that can unexpectedly execute or alter external workflows;
- Python bindings and packaging;
- CI/release infrastructure;
- dependency or supply-chain handling.

Ordinary parser correctness bugs, unsupported syntax and crashes on trusted development inputs can normally be reported through the public bug template unless disclosure would create a security risk.

## Supported versions

EDACurry is currently undergoing repository modularization and is pre-1.0. Until formal release support windows are published, security fixes are expected to target the latest actively maintained development line.

Legacy repositories may remain available for historical reference after migration but should not be assumed to receive security fixes unless explicitly documented.

## Disclosure

Maintainers will coordinate disclosure after the issue has been assessed and a fix or mitigation is available where practical. Credit will be given to reporters who want attribution, subject to their preferred disclosure terms.
