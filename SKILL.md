---
name: code-intel
description: "Use when you need to understand how a codebase works, where to add a feature, or how to integrate it with another system. Triggers on: 'how would we add X', 'how does Y work', 'where do we integrate Z', 'walk me through this code', 'where are the extension points', 'what happens when X fails'."
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

If repo found, check for `sg`:
- Run: `command -v sg >/dev/null 2>&1 && echo "sg available" || echo "sg not found"`
- If not found, tell the user: "Note: `sg` (ast-grep) is not installed. Code searches will use `rg`/`grep` instead. Installing `sg` gives syntax-aware structural search — results contain only real code constructs, not matches inside strings or comments, which reduces context noise and improves result quality. See https://ast-grep.github.io/guide/quick-start.html"
- If found, proceed silently.

Proceed to Step 1.

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

1. sg --lang python -p 'def $NAME($$$)' .
   sg --lang python -p 'class $NAME($$$): $$$' .
   → Find function and class definitions (fallback: rg "^def |^class " --type py)
2. sg --lang python -p 'import $MOD' .
   sg --lang python -p 'from $MOD import $$$' .
   → Map module dependencies
3. find . -type f -name "*.py" | xargs wc -l | sort -rn | head -20
   → Identify largest/key modules by line count
```

*JavaScript/Node.js projects:*
```
1. sg --lang js -p 'function $NAME($$$) { $$$}' .
   sg --lang ts -p 'class $NAME { $$$}' .
   → Find function and class definitions (fallback: rg "^function |^class |export " --type js --type ts)
2. sg --lang js -p 'require($MOD)' .
   sg --lang ts -p 'import $$$from $MOD' .
   → Map module dependencies
3. find . -type f \( -name "*.js" -o -name "*.ts" \) | xargs wc -l | sort -rn | head -20
   → Identify largest/key modules by line count
```

*Go projects:*
```
1. sg --lang go -p 'func $NAME($$$) $$$' .
   sg --lang go -p 'type $NAME struct { $$$}' .
   → Find entry points and core types (fallback: rg "^func |^type " --type go)
2. sg --lang go -p 'import ($$$)' .
   → Map package dependencies (fallback: rg "^import" --type go)
3. find . -type f -name "*.go" | xargs wc -l | sort -rn | head -20
   → Identify largest/key files by line count
```

*Rust projects:*
```
1. sg --lang rust -p 'fn $NAME($$$) { $$$}' .
   sg --lang rust -p 'struct $NAME { $$$}' .
   sg --lang rust -p 'trait $NAME { $$$}' .
   → Find entry points and core types (fallback: rg "^fn |^struct |^trait |^impl " --type rust)
2. sg --lang rust -p 'use $$$' .
   → Map crate dependencies (fallback: rg "^use |^extern crate" --type rust)
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

**Source Definition:** "Source" means tool results from this session only — file reads, grep matches, and command output. Framework/language background knowledge is not a source. Claims from background knowledge must be labeled "Note:" in italics. Claims that are neither traceable to a tool result nor explicitly labeled must be omitted.

**Shared registration audit** (falsification search): When a CLI option, decorator, plugin hook, or shared factory function is identified, search for *all* call sites across the entire relevant directory before making claims about which commands or modules use it. Applies to any pattern where a capability is conferred by a shared decorator or registration call. When claiming multiple files share an identical pattern, also verify local variable names — not just structure — match; enumerate any differences explicitly.

**Comment/annotation attribution:** Claims found in code comments, docstrings, or TODO annotations are developer-stated intent, not verified behavior. Label them: "The author notes in a docstring that..." — not as assertions about runtime behavior.

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

[Where users of this code would hook in new behavior — run Extension guidance audit before writing this section]

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

**Extension guidance audit** (falsification search): Before writing any "minimum diff", "files to touch", or "N-line change" guidance:

1. For each file named: verify it contains the relevant symbol (read the file or grep for the symbol).
2. For each symbol to add or modify: grep for all other files that import or reference it.

If the search was not run, write: `"Files identified by tracing; exhaustive search not performed — verify no additional call sites exist before implementing."` Do not state a specific file count without having verified completeness.

**Verify Before Writing Report** — run this silently before writing any section. No output to user.

Determine analysis scope: if more than 20 files were read or more than 10 grep result sets were processed, use spot-check mode (marked below); otherwise run full verification.

**Full verification:**

1. **Per-claim traceability:** For every file path, function name, line number, call direction ("A calls B"), and dependency claim ("X depends on Y") — confirm it appears in a tool result from this session. Not traceable → omit. Verify named entity type: confirm whether the entity is a function, method, class, interface, or module — do not conflate.

2. **High-risk pattern checklist:**
   - Direction inversions: re-read the tool result for any claim asserting "calls X" vs. "called by X" — these flip easily
   - Named entity type: verify the exact construct found (function vs. method vs. class vs. interface vs. module)
   - Scope trigger words: "always", "throughout", "everywhere", "the system" are banned unless the full scope was traced — replace with the actual path traced
   - Entity relationships: only assert a dependency if an import or call site appeared in tool results

3. **Fabricated sequence (zero-tolerance):** "calls", "which causes", "this triggers", "leading to", and equivalents are prohibited without a traced call chain in tool results. If the sequence cannot be traced, describe what was found and drop the causal assertion.

4. **Enumerated list completeness:** If grep returned N results, report N. "Several", "multiple", "a few", "various" are prohibited when the actual count is known from tool results.

5. **Quantitative fabrication:** One call site found → "one call site." Do not generalize single data points to ranges or vague plurals.

6. **Traceability gate (by section):**
   - *Extension/Integration Points:* every item traces to a tool result
   - *Answer to Your Question:* every file/function/flow claim traces to a tool result
   - *Code Architecture:* every module-responsibility or inter-module claim traces to a tool result
   - Framework background knowledge in any section → label "Note:" in italics

7. **Comment/annotation attribution:** Scan report for claims sourced from comments, docstrings, or TODOs. Verify each is labeled as developer-stated, not asserted as verified behavior.

8. **Internal consistency:** Check these pairs for contradictions — source (tool results) is the tiebreaker:
   - Executive Summary vs. Answer to Your Question
   - Code Architecture vs. Answer to Your Question
   - Strengths/Weaknesses vs. Code Architecture

**Spot-check mode** (large analysis scope — >20 files read or >10 grep result sets):
- Run checks 1, 3, 4, and 5 on Extension/Integration Points and Answer to Your Question only
- Run check 8 on Executive Summary vs. Answer only
- Check 6: verify named specifics in Code Architecture only (skip full traceability gate)

---

**Pre-write validation:**

Required sections: Executive Summary, Code Architecture, Answer to Question, Extension Points, Code Reading Guide, Strengths/Weaknesses, Footer.

If any missing, offer: A) abort, B) save with **INCOMPLETE** warning, C) retry.

**Exclusivity gate** (falsification search): Any claim using "only", "solely", "exclusively", "the only", or "just" in a scope-limiting context. Example: `"only handler.py exposes --verbose"` — run `grep -r "\-\-verbose" src/` and cite the count. If that returns multiple matches, the claim is false and must be rewritten. If the search was not run: `"handler.py exposes --verbose (other modules not verified)"`.

**Directory enumeration** (falsification search): Before asserting which files in a directory share or lack a property, enumerate all files first: `ls src/<module>/*.py`, then grep each for the relevant symbol. If the scan was not run, write: "Commands identified by tracing; exhaustive directory scan not performed — additional commands may exist."

**Quality checks:**
- No weasel words ("likely", "probably", "appears to")
- Specific code references (file names, function names)
- Sentences under 20 words
- Recommendations tied to code structure, not project health

## Strict Fidelity Rules

Apply these throughout the interview and the report.

**Search tool policy:** Use `sg` (ast-grep) for all structural searches. Infer `--lang` from file extensions in context. Fall back to `rg`/`grep` only when ast-grep cannot express the pattern; when falling back, state why in the report.

**Zero-Hallucination (factual claims):** Every factual claim — file paths, function names, call directions, dependencies, counts — must trace to a tool result from this session. If information is unavailable or outside your access, write: "Information not found." Do not bridge gaps with inference.

**Falsification:** Before any completeness or exclusivity claim, run a search that would disprove it. Cite the search command and result count inline. If the search was not run, scope the claim to what was actually traced — not stated as established fact.

**Labeled inference (architectural reasoning):** Inference that connects findings beyond direct trace is permitted only when labeled "Inference:" or "For context:". Unlabeled inference is a hallucination.

**Version/spec keyword validity:** Before asserting that a language feature, schema keyword, library method, or API capability is effective or enforced, verify it is supported by the declared version or spec in use. Check the version declaration first (`$schema`, `"version"`, `requires`, import path, etc.), then confirm the feature exists in that version's vocabulary. If a keyword or feature is present in the file but unsupported by the declared version, state it explicitly: "This [keyword/method] is not valid in [declared version] and will be silently ignored — its presence does not enforce the intended behavior." Do not infer behavior from presence alone.

**Setup method completeness:** For any `__init__`, `activate()`, `_setup()`, `initialize()`, or equivalent setup method, read the full body and enumerate ALL calls — do not filter by relevance to the question. Missing a call misstates the initialization contract.

**Conditional guard check:** Before stating a method is always called, check whether the call site has a conditional guard (`if`, `unless`, `?.`, `&&`, etc.). A guarded call is not unconditional — state the condition explicitly.

**Contradiction Flagging:** If you find conflicting data points, do not reconcile them. Present both sides and label them **Conflicting Evidence**.

**Critical Assessment:** Build Strengths/Weaknesses from available data only. Be skeptical, not optimistic.

**Uncertainty Labeling:** If a claim comes from a non-definitive source, prefix it: "Unverified sources suggest..."

**Ambiguity Check:** If the objective or area of interest is still too broad after the interview, stop and ask for more clarity before generating the report.
