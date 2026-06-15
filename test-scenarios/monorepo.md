# Test Scenario: Monorepo

## Repository Characteristics

**Passive scan detects:**
- Multiple manifest files (e.g., `packages/frontend/package.json`, `services/backend/requirements.txt`)
- Workspace configuration (`lerna.json`, `pnpm-workspace.yaml`, or `workspaces:` in root `package.json`)
- Directory structure suggesting multiple projects (`apps/`, `packages/`, `services/`)
- Mixed languages (Python + JavaScript, Go + TypeScript, etc.)

## Expected Interview Questions

After passive scan summary:
> "I can see this is a monorepo with [X] sub-projects across [Y] languages. What's your objective for this analysis?"

**Follow-up probes if vague:**
> "Are you evaluating the entire monorepo architecture, focusing on a specific service, assessing dependency management practices, or something else?"

**Area of interest clarification:**
> "Should recommendations focus on monorepo tooling, inter-service dependencies, build performance, deployment patterns, or another angle?"

## Expected Report Sections

**Tech Stack & Architecture:**
- Identify monorepo structure
- List sub-projects by type
- Note workspace management tool
- Map dependencies between internal packages

**Key Findings:**
- Circular dependency risks (if any)
- Shared dependency version conflicts
- Build tooling complexity

**Recommendations:**
- Should address monorepo-specific concerns
- Dependency hoisting strategies
- Build caching opportunities
- Deployment coordination patterns

## Edge Case: User Wants Single Sub-Project Analysis

If user clarifies "just analyze the backend service", code-intel should navigate to that directory:
> "Should I navigate to `/services/backend/` and analyze only that sub-project, or analyze the entire monorepo?"

If single sub-project: recommend changing CWD and re-running `/code-intel`.
