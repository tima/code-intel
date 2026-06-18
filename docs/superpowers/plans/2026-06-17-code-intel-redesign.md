# Code-Intel Skill Redesign: Strategic to Tactical

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Redesign code-intel from strategic project analysis to tactical code understanding—enabling developers to learn how code works, where to extend it, and how to integrate it.

**Architecture:** Replace project health assessment (git history, contributor patterns, CI/CD maturity) with code structure analysis (entry points, abstractions, extension points, data flow). Eliminate anti-pattern gates. Reframe interview to elicit code understanding questions. Restructure reports to explain architecture and integration seams instead of sustainability.

**Tech Stack:** Skill written in markdown with embedded shell commands; uses file reading + selective bash execution for code analysis.

## Global Constraints

- No emojis or icons in any output
- Concise language; sacrifice grammar for brevity
- All file paths must be exact
- All code examples in plans must be complete and runnable

---

## File Structure

**Files to modify:**
- `SKILL.md` — core skill definition (rewrite Steps 0-7)
- `README.md` — update positioning and use cases
- `docs/design.md` — update design rationale

**Files to create/replace:**
- `examples/example-django-integration.md` — new example showing code understanding report (replaces Flask example orientation)
- `test-scenarios/code-integration-question.md` — new scenario: "how would we add X"
- `test-scenarios/extension-question.md` — new scenario: "how do we extend Y"
- `test-scenarios/README.md` — update scenario guidance

**Files to remove:**
- `test-scenarios/emergency-request.md`
- `test-scenarios/trivial-repo.md`
- `test-scenarios/validation-seeking.md`

---

### Task 1: Rewrite SKILL.md Step 0 (Anti-Pattern Detection) - Remove Gates

**Files:**
- Modify: `SKILL.md:1-37`

**Interfaces:**
- Produces: Simplified anti-pattern check that validates git repo exists but doesn't block analysis

**Steps:**

- [ ] **Step 1: Replace anti-pattern section with basic validation**

Replace lines 1-37 with:

```markdown
---
name: code-intel
description: Analyze codebases to understand how they work and how to extend/integrate them. Traces code architecture, entry points, extension points, and integration seams. Use to answer "how would we add X", "how does Y work", "where do we integrate Z".
---

---

### Step 0: Repo Validation

Code-intel requires a git repository or recognizable project structure.

**Check for repo presence:**
- Run: `test -d .git && echo "git repo found" || echo "no git repo"`
- Check for manifest files: `package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`, `build.gradle`, `Gemfile`, `composer.json`

If no `.git/` and no manifest files found, stop with message:

> "I don't see a recognizable code repository here. Navigate to a project root and try again."

If repo found, proceed to Step 1.

---
```

- [ ] **Step 2: Commit the change**

```bash
git add SKILL.md
git commit -m "refactor: replace anti-pattern gates with simple repo validation"
```

---

### Task 2: Rewrite SKILL.md Step 1-2 (Passive Scan) - Focus on Code Structure

**Files:**
- Modify: `SKILL.md:40-135`

**Interfaces:**
- Consumes: Repo validation from Task 1
- Produces: Code structure data: entry points, core abstractions, extension mechanisms, public APIs, config points

**Steps:**

- [ ] **Step 1: Replace manifest/infrastructure scan with code structure scan**

Replace lines 40-135 with:

```markdown
### Step 1: Identify Repository Language & Type

Read any README file (`README.md`, `README`, `README.rst`) to understand project purpose. Check manifest files to determine primary language and framework.

---

### Step 2: Passive Code Structure Scan

Use file reading and pattern matching only. Do NOT run shell commands at this stage.

**Identify entry points (where code starts):**
1. Look for: `main()` functions, `if __name__ == "__main__"`, server startup files (`app.py`, `server.py`, `index.js`, `main.rs`, `main.go`)
2. Check `package.json` for `"main"`, `"bin"`, or `"scripts"` fields
3. Check for CLI entry points in manifest (`entry_points` in `setup.py`, `[bin]` in `Cargo.toml`)

**Identify core abstractions:**
4. Search for abstract base classes or interfaces (files named `base.py`, `abstract.py`, `interface.ts`, traits in Rust)
5. Scan for key class definitions and their inheritance hierarchy
6. Identify major modules and their responsibility

**Identify extension/integration points:**
7. Look for patterns: decorators, hooks/callbacks, middleware, factory methods, plugin systems
8. Check for configuration-driven behavior (`__init__` parameters, config files, env var reading)
9. Scan for callback registration (`.on()`, `.register()`, `.subscribe()`, event listeners)

**Identify public APIs:**
10. Check which functions/classes are exported (top-level in main files, `__all__` in Python, `export` statements)
11. Identify main interfaces users would interact with

**From this scan, document:**
- Entry point(s) and startup sequence
- Core abstractions and how they relate
- Extension mechanisms (how users hook in)
- Configuration options and where they're read
- What's public vs internal

---
```

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "refactor: rewrite passive scan to identify code structure instead of project health"
```

---

### Task 3: Rewrite SKILL.md Step 3 (Interview) - Elicit Tactical Code Questions

**Files:**
- Modify: `SKILL.md:136-175`

**Interfaces:**
- Consumes: Code structure data from Task 2
- Produces: Clear tactical question and scope for analysis

**Steps:**

- [ ] **Step 1: Replace context scoring with direct code question**

Replace lines 136-175 with:

```markdown
### Step 3: Interview

Present a brief 1-sentence summary of what you found in the passive scan. Then ask one direct question:

**Example opener:**

> "I can see this is a Python FastAPI service with database models and async task handling. What specifically do you want to understand about this code?"

**Probe for specificity. Valid answers include:**
- "How would we add authentication to this?"
- "How does the data flow from request to database?"
- "Where would we integrate Kafka for event processing?"
- "How do we extend this to support custom plugins?"
- "What happens when X fails and where is that logic?"

**If vague, ask:**
> "Are you trying to understand how a specific feature works, evaluate how to add something new, or figure out where to hook in external integrations?"

Keep asking clarifying questions until the user's question is specific enough that you could trace code to answer it.

**Proceed to Step 4** when you understand: what part of the code they want to understand, and what operation/change they're evaluating.

---
```

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "refactor: replace context scoring with direct tactical code questions"
```

---

### Task 4: Rewrite SKILL.md Steps 4-5 (Permissions & Deep Analysis) - Code Tracing

**Files:**
- Modify: `SKILL.md:176-242`

**Interfaces:**
- Consumes: Tactical code question from Task 3
- Produces: List of code exploration commands (grep, trace, inspect) tailored to the question

**Steps:**

- [ ] **Step 1: Replace shell command gate with code-tracing commands**

Replace lines 176-242 with:

```markdown
### Step 4: Permission Gate

Before running code analysis commands, present what you plan to run and ask for permission.

**Propose stack-specific code exploration commands:**

```
To trace the code paths related to your question, I'd like to run:

1. grep -rn "def main\|def __init__\|class.*:" --include="*.py" | head -30
   → Find entry points and core classes
2. grep -rn "import\|from .* import" --include="*.py" | head -50
   → Map module dependencies
3. find . -type f -name "*.py" | xargs wc -l | sort -rn | head -20
   → Identify largest/key modules by line count

Options:
A) Approve all
B) Approve individually
C) Decline all (we'll work from file inspection only)

Your choice?
```

If user chooses C, note in report: "Analysis based on file reading only (no code tracing)."

---

### Step 5: Deep Analysis

Run approved commands. For the user's specific question, trace code paths:

**Example trace for "how would we add authentication?":**
- Find where requests enter the system (entry point from Step 2)
- Grep for `def get_user()`, `auth`, `token` patterns
- Identify existing auth/permission checks
- Trace from entry point → auth check → user context
- Identify where user context is passed/stored
- Document what an auth implementation would need to hook into

**Apply only to the user's specific question.** Don't do comprehensive analysis of everything — focus the trace on their code understanding goal.

---
```

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "refactor: replace dependency audit with code-tracing commands focused on user question"
```

---

### Task 5: Rewrite SKILL.md Steps 6-7 (Report Structure) - Code Understanding Focus

**Files:**
- Modify: `SKILL.md:243-304`

**Interfaces:**
- Consumes: Deep analysis traces from Task 4
- Produces: Structured markdown report explaining code architecture and answering tactical question

**Steps:**

- [ ] **Step 1: Replace report structure entirely**

Replace lines 243-304 with:

```markdown
### Step 6: Get Timestamp

Capture current date/time for report footer:

Run: `date -u +"%b-%d-%Y %H:%M UTC"` or use current date if unavailable.

Store the timestamp — do not re-fetch.

---

### Step 7: Generate Report

Create a report answering the user's tactical code question. Save to CWD as: `code-intel-<project-name>-<YYYY-MM-DD>.md`

**Report structure:**

```markdown
# [Project Name] — Code Intelligence Report

**Objective:** [User's specific question]  
**Repository:** [CWD]

---

## Executive Summary

[1-2 sentences answering their question directly. Example: "Authentication is currently bypassed in development mode; to add production auth, you'd implement the `AuthProvider` interface and pass it to `Application.__init__()`."]

---

## Code Architecture

[How the code is organized. Example: "Core request dispatch lives in `app.py`. Middleware chain in `middleware/`. Database models in `models/`. Auth currently only checks a hardcoded role."]

[Entry points: where execution starts]  
[Key modules and their responsibility]  
[How modules interact]

---

## Answer to Your Question

[Directly answer "how would we add X" / "how does Y work" / "where do we integrate Z"]

[Trace the code path relevant to their question. Include actual file names and function/class names.]

Example format for "how would we add authentication":
- Request enters via `app.py:handle_request()`
- Passes through middleware in `middleware/chain.py`
- Calls handler in `handlers/auth.py:verify_user()`
- Currently returns hardcoded user object from line 23
- To add real auth: implement `AuthProvider` interface (defined in `auth/base.py`), pass to `Application.__init__()` at line 15

---

## Extension/Integration Points

[Where users of this code would hook in new behavior]

- To add custom logging: subclass `LogHandler` in `logging/base.py` and register in `app.py:setup_logging()`
- To add database migrations: place files in `migrations/` with pattern `00N_*.sql`
- To extend API responses: implement `ResponseFormatter` interface in `formatters/`

---

## Code Reading Guide

[Suggest the order to understand the code]

1. Start with `app.py` to understand initialization
2. Read `handlers/` to see request handling
3. Understand middleware chain in `middleware/chain.py`
4. Examine `models/` for data structures

---

## Strengths & Weaknesses

| Strengths | Weaknesses |
|-----------|-----------|
| [2+ code-specific strengths with evidence] | [2+ code-specific weaknesses with evidence] |

Example: "Clean separation between data models and request handlers" | "Request handler does too much; should split validation from dispatch"

---

[Provider/Tool] | [Model] | [Timestamp UTC]
```

**Pre-write validation:**

Required sections: Executive Summary, Code Architecture, Answer to Question, Extension Points, Code Reading Guide, Strengths/Weaknesses, Footer.

If any missing, offer: A) abort, B) save with **INCOMPLETE** warning, C) retry.

**Quality checks:**
- No weasel words ("likely", "probably", "appears to")
- Specific code references (file names, function names)
- Sentences under 20 words
- Recommendations tied to code structure, not project health

---
```

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "refactor: redesign report structure for code understanding instead of project assessment"
```

---

### Task 6: Create New Example Report — Django Integration Question

**Files:**
- Create: `examples/example-django-integration.md`

**Interfaces:**
- Consumes: New report structure from Task 5
- Produces: Complete example showing code understanding output for typical question

**Steps:**

- [ ] **Step 1: Write example report**

Create file with content:

```markdown
# Django REST Framework — Code Intelligence Report

**Objective:** How would we add custom request validation middleware?  
**Repository:** /tmp/djangorestframework

---

## Executive Summary

Custom validation middleware hooks into DRF's middleware chain via the `APIView.dispatch()` method. To add request validation, you'd create a custom middleware class inheriting from `BaseMiddleware` in `rest_framework/middleware.py` and register it in Django's `MIDDLEWARE` setting.

---

## Code Architecture

- **Entry point:** Django WSGI handler → `rest_framework/views.py:APIView.dispatch()`
- **Middleware chain:** `rest_framework/middleware.py` contains `BaseMiddleware`, `APIMiddleware`
- **Request processing:** Validation happens in `decorators.py` (for function views) or `APIView.initial()` (for class views)
- **Exception handling:** `rest_framework/exceptions.py` defines all validation exceptions
- **Permissions:** `rest_framework/permissions.py` has permission classes that run during `APIView.check_permissions()`

**Key modules:**
- `views.py` — APIView base class, request dispatch
- `middleware.py` — middleware base and authentication middleware
- `decorators.py` — function-based view decorators
- `exceptions.py` — validation and error classes
- `permissions.py` — permission checking logic

---

## Answer to Your Question

**How to add custom request validation middleware:**

1. Create middleware class in `your_app/middleware.py`:

```python
from rest_framework.middleware import BaseMiddleware

class CustomValidationMiddleware(BaseMiddleware):
    def process_request(self, request):
        # Your validation logic here
        if not self.is_valid(request):
            raise ValidationError("Invalid request")
        return request
```

2. Where it hooks in:

   - `rest_framework/views.py:APIView.dispatch()` (line 487) calls `self.middleware_chain(request)`
   - Middleware chain defined in `settings.MIDDLEWARE`
   - Your middleware's `process_request()` runs before handlers

3. Request flow:

   Django WSGI
   → Django middleware stack (`settings.MIDDLEWARE`)
   → Your `CustomValidationMiddleware.process_request()`
   → `APIView.dispatch()` → `APIView.initial()` (permission checks, versioning, etc.)
   → Handler (GET/POST/etc.)

4. To register:

   Add to `settings.py`:
   ```python
   MIDDLEWARE = [
       ...
       'your_app.middleware.CustomValidationMiddleware',
   ]
   ```

---

## Extension/Integration Points

- **Add field validation:** Subclass `Serializer` in `serializers.py:BaseSerializer`, override `validate_<field>()`
- **Add custom permissions:** Subclass `BasePermission` in `permissions.py` and implement `has_permission()`
- **Add request preprocessing:** Create middleware inheriting `BaseMiddleware`, hook in `process_request()`
- **Add custom exceptions:** Create exception classes in `exceptions.py` inheriting from `APIException`

---

## Code Reading Guide

1. Start: `views.py:APIView` — understand request dispatch entry point
2. Trace: `APIView.dispatch()` → `APIView.initial()` to see where permissions/validation run
3. Study: `middleware.py` to understand middleware chain integration
4. Examine: `permissions.py` and `decorators.py` for validation patterns
5. Reference: `exceptions.py` for available error types

---

## Strengths & Weaknesses

| Strengths | Weaknesses |
|-----------|-----------|
| Clear separation: APIView.dispatch() has defined hook points (initial(), permission_classes) | Permission checking tied to APIView; function-based views need decorators, adding friction |
| Middleware pattern familiar to Django devs | BaseMiddleware inheritance required; could be simpler for basic cases |
| Exception classes follow hierarchy; easy to catch/handle specific errors | Exception hierarchy spans two files (exceptions.py, status.py); harder to discover |

---

[Provider/Tool] | [Model] | [Timestamp UTC]
```

- [ ] **Step 2: Commit**

```bash
git add examples/example-django-integration.md
git commit -m "docs: add example code understanding report for Django integration question"
```

---

### Task 7: Create New Test Scenarios

**Files:**
- Create: `test-scenarios/code-integration-question.md`
- Create: `test-scenarios/extension-question.md`
- Delete: `test-scenarios/emergency-request.md`, `test-scenarios/trivial-repo.md`, `test-scenarios/validation-seeking.md`
- Modify: `test-scenarios/README.md`

**Interfaces:**
- Consumes: Interview flow from Task 3
- Produces: New test scenarios for typical code understanding questions

**Steps:**

- [ ] **Step 1: Create code-integration-question scenario**

Create `test-scenarios/code-integration-question.md`:

```markdown
# Test Scenario: Code Integration Question

**User asks:** "How would we integrate Kafka into this FastAPI app?"

**Expected flow:**

1. Repo validation passes (FastAPI app present)
2. Passive scan identifies: entry point in `main.py`, async task handling in `celery_tasks.py`
3. Interview asks: "What specifically do you want to understand about this code?"
4. User clarifies: "Where would we add event publishing to Kafka instead of Celery?"
5. Analysis traces: event publication points → current Celery integration → where Kafka producer would hook in
6. Report answers: "Event publishing happens in `services/events.py:publish_event()`. Currently calls Celery task. To add Kafka, you'd implement `EventPublisher` interface and pass to `Application` constructor, similar to how `CeleryPublisher` is wired at line 42."

**Success criteria:**
- No anti-pattern gates block the request
- Interview produces a specific code understanding goal
- Report directly answers the integration question
- Report includes exact file names and function names
```

- [ ] **Step 2: Create extension-question scenario**

Create `test-scenarios/extension-question.md`:

```markdown
# Test Scenario: Extension/Customization Question

**User asks:** "How would we add custom validation rules to this form handler?"

**Expected flow:**

1. Repo validation passes (Python web app present)
2. Passive scan identifies: form handling in `forms.py`, validation in `validators.py`
3. Interview asks: "What specifically do you want to understand about this code?"
4. User clarifies: "Where would we hook in custom field-level validators?"
5. Analysis traces: form submission → validation chain → current validator architecture
6. Report answers: "Validation happens in `validators.py:validate_form()`. Current validators in `VALIDATORS` dict. To add custom validators: create `CustomValidator` class inheriting `BaseValidator`, implement `validate()` method, register in `VALIDATORS` dict at line 78."

**Success criteria:**
- Report includes exact extension points (where to add code)
- Report shows code structure supporting customization
- Report provides minimal code example of how to extend
```

- [ ] **Step 3: Update test-scenarios README**

Modify `test-scenarios/README.md`:

```markdown
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
```

- [ ] **Step 4: Delete old scenarios**

```bash
rm test-scenarios/emergency-request.md test-scenarios/trivial-repo.md test-scenarios/validation-seeking.md
git add test-scenarios/
git commit -m "refactor: replace anti-pattern scenarios with code understanding scenarios"
```

---

### Task 8: Update README.md and Design Documentation

**Files:**
- Modify: `README.md`
- Modify: `docs/design.md`

**Interfaces:**
- Consumes: New skill behavior from Tasks 1-7
- Produces: Updated user-facing and design documentation

**Steps:**

- [ ] **Step 1: Rewrite README positioning**

Replace README.md content with:

```markdown
# code-intel

Analyze codebases to understand how they work and how to extend/integrate them.

## What It Does

Acts as a code analyst who interviews you about your specific question, traces relevant code paths, and produces a structured markdown report explaining architecture, entry points, extension points, and integration seams.

Reports are laser-focused on answering your tactical code question — not project health or sustainability.

## When to Use

**Good fit:**
- "How would we add authentication?"
- "How does the payment flow work?"
- "Where would we integrate Kafka?"
- "How do we extend this with plugins?"
- "What happens when X fails?"

**When NOT to use:**
- You need to fix a specific bug (use debugging, not analysis)
- You need a quick security audit (use tool-specific audits: `npm audit`, `cargo audit`)
- You're evaluating project health/team sustainability (no longer what code-intel does)

## Installation

```bash
git clone https://github.com/username/code-intel-skill.git ~/projects/code-intel-skill
ln -s ~/projects/code-intel-skill ~/.claude/skills/code-intel
```

## Usage

Navigate to a code repository and invoke:

```
/code-intel
```

The skill will:
1. Verify you're in a code repository
2. Scan code structure (entry points, abstractions, extension mechanisms)
3. Ask what you want to understand about the code
4. Trace relevant code paths
5. Generate and save a report

Reports are saved as `code-intel-<project-name>-<YYYY-MM-DD>.md`.

## Example Output

See [example Django integration analysis](examples/example-django-integration.md) for a sample report structure.

## Requirements

- Git repository or recognizable project structure (package.json, requirements.txt, go.mod, Cargo.toml, etc.)

## Documentation

- [Design Doc](docs/design.md) - Design rationale and architecture
- [Examples](examples/) - Sample reports

## License

MIT
```

- [ ] **Step 2: Update design.md**

Modify `docs/design.md` to replace the Strategic Analysis rationale with:

```markdown
# Code-Intel Design

## Purpose

Code-intel helps developers understand how code works and how to extend/integrate it by tracing code structure and explaining architectural entry/extension points.

## NOT Strategic Analysis

Earlier versions positioned code-intel as a strategic tool for project health assessment. This approach had low product-market fit:

- Narrow audience (PMs, CTOs evaluating acquisitions)
- Competed with existing project auditing tools
- Required long analysis time for tangential use case
- Anti-pattern gates blocked legitimate uses

Redesigned for tactical code understanding — the actual use case our engineers had.

## Core Workflow

1. **Repo validation** — Confirm git repo or manifest
2. **Code structure scan** — Identify entry points, abstractions, extension patterns (file reading only)
3. **Tactical question** — Interview to establish what code understanding goal
4. **Code tracing** — Run approved commands to trace relevant paths
5. **Focused report** — Answer the question with code references and extension guidance

## Key Design Decisions

**No anti-pattern gates.** Removed emergency/trivial/validation-seeking gates. Users know their own use case; blocking wastes their time.

**Passive scan finds structure, not health.** Look for entry points, abstractions, extension mechanisms — not CI/CD, security practices, or contributor patterns.

**Interview drives scope.** Don't analyze everything; trace only the code paths relevant to the user's question.

**Reports answer the question directly.** First section: executive summary that directly answers "how would we add X?" Subsequent sections provide the evidence.

**No jargon, no generalization.** Cite exact file names, function names, line numbers. No "this framework is well-designed" — say "validation happens in `validators.py:validate_form()`."

## Architecture

See SKILL.md for step-by-step implementation details.

## Future

Potential enhancements:
- Generate code snippets showing minimal examples of extensions
- Trace performance-critical paths when asked about performance
- Suggest refactoring when code understanding reveals maintainability issues (out of scope for MVP)
```

- [ ] **Step 3: Commit both files**

```bash
git add README.md docs/design.md
git commit -m "docs: reposition code-intel as tactical code understanding tool"
```

---

### Task 9: Validate with Test Run

**Files:**
- Use existing test scenarios from Task 7

**Interfaces:**
- Consumes: Complete redesigned skill from Tasks 1-8
- Produces: Verification that skill runs without errors on a real codebase

**Steps:**

- [ ] **Step 1: Pick a test repository**

Use a small, well-structured codebase. Example options:
- Flask (`git clone https://github.com/pallets/flask`)
- FastAPI (`git clone https://github.com/tiangolo/fastapi`)
- Or any local project you have

- [ ] **Step 2: Run skill with code integration question**

```bash
cd /path/to/test/repo
/code-intel
```

When prompted for question, answer with code understanding question:
Example: "How would we add request logging middleware?"

- [ ] **Step 3: Verify report output**

Check that generated report:
- Has all required sections (Executive Summary, Code Architecture, Answer to Question, Extension Points, Code Reading Guide, Strengths/Weaknesses)
- Directly answers the user's question in Executive Summary
- References exact file names and function names
- Includes no weasel words ("likely", "probably")
- Has correct footer with provider/model/timestamp

- [ ] **Step 4: Test with extension question**

Run skill again with different question:
Example: "How do we add custom error handlers?"

Verify report structure and accuracy.

- [ ] **Step 5: Commit validation notes**

If any issues found, file them as separate tasks. If all pass:

```bash
git add .
git commit -m "test: validate redesigned skill with real codebase"
```

---

### Task 10: Final Documentation Review

**Files:**
- Review: `SKILL.md` (steps, commands, examples)
- Review: `README.md` (positioning, examples)
- Review: `examples/` (report structure matches documented output)
- Review: All error messages and prompts (no weasel words, clear language)

**Interfaces:**
- Consumes: All changes from Tasks 1-9
- Produces: Clean, complete documentation ready for deployment

**Steps:**

- [ ] **Step 1: Review SKILL.md for clarity and completeness**

Scan for:
- Every command is exact and runnable
- No "TBD" or "fill in" placeholders
- All code examples are complete
- Interview prompts are clear
- Report sections have concrete guidance

- [ ] **Step 2: Review README for user clarity**

Verify:
- Use cases are clear and specific
- Installation instructions work
- Example report is relevant to new use case
- No references to old strategic analysis positioning

- [ ] **Step 3: Verify example report matches output structure**

Check `examples/example-django-integration.md`:
- All required sections present
- Code references are specific
- Extension points clearly marked
- Reading guide is helpful for new reader
- No jargon or vague language

- [ ] **Step 4: Check all prompts and error messages**

Scan full text of SKILL.md for user-facing text:
- Questions are clear and specific
- Error messages explain what went wrong
- No condescending language
- Language is concise (no padding)

- [ ] **Step 5: Final commit**

```bash
git add .
git commit -m "docs: final review of redesigned code-intel skill"
```
