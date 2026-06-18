# Test Scenario: Monorepo

## Repository Characteristics

**Passive scan detects:**
- Multiple manifest files (e.g., `packages/frontend/package.json`, `services/backend/requirements.txt`)
- Workspace configuration (`lerna.json`, `pnpm-workspace.yaml`, or `workspaces:` in root `package.json`)
- Directory structure suggesting multiple projects (`apps/`, `packages/`, `services/`)
- Mixed languages (Python + JavaScript, Go + TypeScript, etc.)

## Expected Interview

After passive scan summary, the skill asks one direct question:

> "I can see this is a monorepo with [X] sub-projects across [Y] languages. What specifically do you want to understand about this code?"

**Valid specific answers:**
- "How would we add a new service that integrates with the payment system?"
- "Where would we hook in a shared logging library across all services?"
- "How do dependencies flow between the frontend and backend services?"
- "Where is the deployment orchestration and how would we add a new service to it?"

**If vague** (e.g., "understand the architecture"), probe:
> "Are you trying to understand how a specific service works, see how two services interact, or figure out where to add a new integration?"

Keep probing until the question identifies a specific code path, feature, or integration point.

## Expected Report Sections

**Code Architecture:**
- Monorepo structure: sub-project list, workspace tool (lerna, pnpm, Turborepo)
- Entry points per service
- Inter-service dependency map (which packages import which)

**Answer to Question:**
- Traced code path answering the specific question
- Exact file/function names in the relevant service(s)

**Extension/Integration Points:**
- Where new services plug into shared infrastructure
- How cross-service contracts are defined (shared types, API clients, event schemas)

## Edge Case: User Wants Single Sub-Project Analysis

If user clarifies "just analyze the backend service", code-intel should navigate to that directory:
> "Should I navigate to `/services/backend/` and analyze only that sub-project, or analyze the entire monorepo?"

If single sub-project: recommend changing CWD and re-running `/code-intel`.
