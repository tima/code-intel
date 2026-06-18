# Test Scenarios

These scenarios test code-intel's ability to handle typical user questions about code understanding and modification.

## Scenarios

- **code-integration-question** — User wants to integrate external system (Kafka, Redis, etc.)
- **extension-question** — User wants to extend behavior (custom validators, plugins, etc.)
- **monorepo** — Repository with multiple projects/packages (existing scenario, kept as-is)

## Removed Scenarios

- emergency-request — No longer blocks analysis
- trivial-repo — No longer blocks analysis
- validation-seeking — No longer blocks analysis

These patterns were gatekeeping strategic misuse. Code-intel now focuses on tactical code understanding, which these restrictions don't apply to.
