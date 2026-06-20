---
name: code-intel
description: Analyze codebases to understand how they work and how to extend/integrate them. Traces code architecture, entry points, extension points, and integration seams. Use to answer "how would we add X", "how does Y work", "where do we integrate Z".
---

# code-intel

Analyze codebases to understand how they work and how to extend or integrate them.

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

### Step 1: Identify Repository Language & Type

Read any README file (`README.md`, `README`, `README.rst`) to understand project purpose. Check manifest files to determine primary language and framework.

---

### Step 2: Passive Code Structure Scan

Use file reading and pattern matching only. Do NOT run shell commands at this stage.

**Identify entry points (where code starts):**

Adapt to detected language from Step 1:

*Python:* `if __name__ == "__main__"`, files named `main.py`/`app.py`/`server.py`; `entry_points`/`console_scripts` in `setup.py`; `[project.scripts]` in `pyproject.toml`

*JavaScript/Node.js:* `index.js` or file referenced in `package.json:"main"`; `"scripts":"start"` in `package.json`; top-level `app.listen()`/`server.listen()`

*Go:* `func main()` in any `package main` file; `cmd/` directory containing main packages; check `go.mod` for module name

*Rust:* `fn main()` in `src/main.rs`; `[[bin]]` entries in `Cargo.toml`

*Ruby:* `bin/` directory; files requiring `./config/application` (Rails); `Rakefile` entry tasks

*Java/Kotlin:* Classes with `public static void main`; Spring `@SpringBootApplication`; `pom.xml`/`build.gradle` mainClass config

1. Find entry point file(s) using patterns above
2. Check `package.json` for `"main"`, `"bin"`, or `"scripts"` fields (all JS/Node projects)
3. Check manifest for CLI entry points

**Identify core abstractions:**
4. Search for abstract base classes or interfaces (Python: `base.py`, `abstract.py`; TypeScript: `interface.ts`; Go: interfaces in `*.go`; Rust: traits; Java: `Abstract*.java`, `*Interface.java`)
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

**Stop asking when the question meets all three criteria:**
1. Identifies a specific code path, feature, or integration point
2. Is answerable by reading code and tracing execution
3. Can be demonstrated with exact file/function names in the answer

*Specific enough:*
- "How does the payment processing flow work from API call to database write?"
- "Where would we hook in custom authentication middleware?"
- "What happens when a webhook delivery fails and where is the retry logic?"

*Too vague (ask again):*
- "How does this work?" — which part?
- "Help me understand the codebase" — no specific target
- "Is this any good?" — evaluating health, not understanding code

**Proceed to Step 4** when the question is specific enough to trace.

---

### Step 4: Permission Gate

Before running code analysis commands, present what you plan to run and ask for permission.

**Propose stack-specific code exploration commands:**

Adapt commands to the language detected in Step 1.

*Python projects:*
```
To trace the code paths related to your question, I'd like to run:

1. grep -rn "def main\|def __init__\|class.*:" --include="*.py" | head -30
   → Find entry points and core classes
2. grep -rn "^import\|^from .* import" --include="*.py" | head -50
   → Map module dependencies
3. find . -type f -name "*.py" | xargs wc -l | sort -rn | head -20
   → Identify largest/key modules by line count
```

*JavaScript/Node.js projects:*
```
1. grep -rn "function \|class \|module.exports\|export " --include="*.js" --include="*.ts" | head -30
   → Find entry points and core classes
2. grep -rn "require(\|import " --include="*.js" --include="*.ts" | head -50
   → Map module dependencies
3. find . -type f \( -name "*.js" -o -name "*.ts" \) | xargs wc -l | sort -rn | head -20
   → Identify largest/key modules by line count
```

*Go projects:*
```
1. grep -rn "func \|type.*struct\|interface {" --include="*.go" | head -30
   → Find entry points and core types
2. grep -rn "^import" --include="*.go" | head -50
   → Map package dependencies
3. find . -type f -name "*.go" | xargs wc -l | sort -rn | head -20
   → Identify largest/key files by line count
```

*Rust projects:*
```
1. grep -rn "fn \|struct \|trait \|impl " --include="*.rs" | head -30
   → Find entry points and core types
2. grep -rn "^use \|^extern crate" --include="*.rs" | head -50
   → Map crate dependencies
3. find . -type f -name "*.rs" | xargs wc -l | sort -rn | head -20
   → Identify largest/key modules by line count
```

Present the adapted command list, then:

```
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

**If code tracing fails or yields no results:**

1. Fall back to manual file inspection (read key files identified in Step 2)
2. If still unable to trace the path, document in report under **Analysis Limitations**:
   > "Could not trace [specific path] — [reason: pattern not found / code is dynamically generated / insufficient signal in static analysis]. Analysis based on [files inspected]."
3. Report what was found; don't fill gaps with inference

Never hallucinate code paths. If evidence is absent, say so.

---

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

Claude Code | [Model: e.g. "Haiku 4.5" or "Sonnet 4.6"] | [Timestamp from Step 6]
```

**Pre-write validation:**

Required sections: Executive Summary, Code Architecture, Answer to Question, Extension Points, Code Reading Guide, Strengths/Weaknesses, Footer.

If any missing, offer: A) abort, B) save with **INCOMPLETE** warning, C) retry.

**Quality checks:**
- No weasel words ("likely", "probably", "appears to")
- Specific code references (file names, function names)
- Sentences under 20 words
- Recommendations tied to code structure, not project health

## Strict Fidelity Rules

Apply these throughout the interview and the report.

**Zero-Hallucination:** If information is unavailable or outside your access, write: "Information not found." Never bridge gaps with logical inferences or "likely" scenarios.

**Contradiction Flagging:** If you find conflicting data points, do not reconcile them. Present both sides and label them **Conflicting Evidence**.

**Critical Assessment:** Build Strengths/Weaknesses from available data only. Be skeptical, not optimistic.

**Uncertainty Labeling:** If a claim comes from a non-definitive source, prefix it: "Unverified sources suggest..."

**Ambiguity Check:** If the objective or area of interest is still too broad after the interview, stop and ask for more clarity before generating the report.
