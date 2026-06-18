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

### Step 3: Interview

Present a brief 1–2 sentence context summary of what you found in the passive scan. Then ask clarifying questions **one at a time** to establish:

1. **Objective** — what the user wants to learn or achieve
2. **Area of Interest** — the focus angle for the recommendations section

Keep asking follow-up questions if answers are vague, too broad, or ambiguous. Do NOT proceed to Step 4 until both the objective and the area of interest are specific enough to produce a focused report.

**Example opener:**

> "I can see this is a Python FastAPI service with PostgreSQL and Celery for async tasks. What's your objective for this analysis?"

**If the answer is vague:**

> "Are you evaluating it for adoption, looking for risks before a release, assessing maintainability, or something else?"

**Example area-of-interest prompt:**

> "Got it. What angle should I focus recommendations on? For example: security posture, dependency health, code maintainability, architectural fitness, contributor health, or release readiness?"

Keep probing until both are specific. If the objective and area of interest remain too broad after follow-up, stop and ask again before moving on.

A good objective names a concrete deliverable — for example: "decide whether to adopt this library," "prepare a risk briefing for the upcoming release," or "assess whether this codebase can support the new feature." If the user's answer is still a general verb or stance without a concrete output, ask again.

A good area of interest names a specific lens — for example: "dependency health," "security posture," "release readiness," or "contributor sustainability." If the answer is too broad (e.g., "everything" or "general quality"), ask the user to pick one focus area.

**Context Scoring:**

Collect 5 items: objective, area of interest, role, timeline, success criteria

- **5/5** = proceed to Step 4
- **3-4/5** = warn user, list missing items, ask to proceed with caveats or provide more context
- **0-2/5** = STOP, list missing items with rationale, offer: A) generic overview (no Recommendations), B) continue clarification
- **>5 rounds at <3/5** = offer to stop (objective may be unclear)

If user stops or proceeds with <3/5, note `**Context Limitations:**` in report.

Only proceed to Step 4 if score is 3/5 or higher.

---

### Step 4: Permission Gate with Selective Approval

Before running any shell commands, present a list of exactly what you plan to run and ask for permission. Tailor the command list to the detected tech stack.

**Present commands with individual approval options:**

```
To do a thorough analysis, I'd like to run the following commands.

Approve all, approve individually, or decline all?

Proposed commands:
1. `git log --oneline -50` — recent commit history
2. `git shortlog -sn --no-merges` — contributor activity  
3. `git diff HEAD~10..HEAD --stat` — recent change volume
4. `npm audit --json` — dependency vulnerability check (Node.js detected)

Options:
A) Approve all
B) Approve individually (I'll ask for each)
C) Decline all (passive scan only)

Your choice?
```

**If user chooses A (approve all):** Run all commands.

**If user chooses B (approve individually):** For each command, ask:
```
Run `[command]` — [description]? (y/n)
```
Run only approved commands.

**If user chooses C (decline all):** Proceed with passive scan data only and note in report:
```
**Analysis Limitations:** User declined shell command permissions. Analysis based on passive file scan only (no git history, no dependency audit results).
```

**Track which commands were approved** for use in report limitations section.

**Stack-specific dependency audit commands — include when relevant:**

| Stack | Command |
|---|---|
| Node.js | `npm audit --json` or `yarn audit --json` |
| Python | `pip-audit` or `safety check` |
| Rust | `cargo audit` |
| Ruby | `bundle audit` |
| Go | `govulncheck ./...` |
| Java/Maven | `mvn dependency:analyze` |

---

### Step 5: Deep Analysis

Run all approved commands and synthesize the findings. Apply only the areas that are relevant to the stated objective and the tools the user approved.

- **Git history** — commit cadence, contributor spread, recent activity, areas of churn
- **Dependency health** — outdated packages, known vulnerabilities, dependency count
- **Code structure, tests, and CI/CD** — use file reading and pattern matching (not shell commands) to explore directory organization, size distribution, test coverage, and pipeline config. This applies even if the user declined some or all shell commands.
- **Configuration and environment** — `.env` files, config patterns, secrets hygiene signals

---

### Step 6: Get Timestamp

Capture the current UTC time ONCE and store it for use in Step 7. This prevents time drift between capture and report writing.

**Timestamp capture priority order:**

1. **Shell command (preferred):** Run `date -u +"%b-%d-%Y %H:%M UTC"`
2. **Web search (fallback):** Search "current UTC time" and extract the result.
3. **Last resort:** Use `[timestamp unavailable]`.

**Store the result immediately.** Do not re-fetch the timestamp in Step 7. Use the captured value in the report footer.

**Example:**
- Capture: `Jun-15-2026 18:32 UTC`
- Use in footer: `Claude Code | Claude Sonnet 4.6 | Jun-15-2026 18:32 UTC`

---

### Step 7: Generate and Save Report

Generate the full report, then save it to the CWD.

**Filename format:** `code-intel-<project-name>-<YYYY-MM-DD>.md`

- Derive `<project-name>` from the CWD directory name or the project name in the manifest. Use lowercase with hyphens for spaces. Keep it under 40 characters.
- `<YYYY-MM-DD>` comes from the timestamp captured in Step 6.
- Check whether that filename already exists. If it does, append `-2`, `-3`, etc. until you find an unused name.

Before writing: announce `"Saving report to <filename>..."`

After writing: confirm `"Report saved to <filename>."`

**Report structure:** See examples/example-flask-library.md for template. Footer format: `[Provider/Tool] | [Model] | [Timestamp UTC]`

---

**Pre-write Validation:**

**Required (fail-fast if missing):** Exec Summary (2-3 sentences, specific finding), Tech Stack (specific languages/frameworks/versions), 3+ Key Findings (not generalizations), 2+ Detailed Analysis subsections, Strengths/Weaknesses table (2+ per column, evidence-based), Recommendations (title includes area of interest, 3+ actionable items), Footer (provider/model/timestamp)

**Quality (warn but proceed):** No weasel words ("likely", "probably"), specific numbers ("139 commits" not "many"), recommendations address objective, <20 word sentences, no jargon

**If required missing:** Offer A) abort, B) save with **INCOMPLETE ANALYSIS** disclaimer, C) retry with more data

**If quality fails:** Warn, list issues, proceed with save

## Strict Fidelity Rules

Apply these throughout the interview and the report.

**Zero-Hallucination:** If information is unavailable or outside your access, write: "Information not found." Never bridge gaps with logical inferences or "likely" scenarios.

**Contradiction Flagging:** If you find conflicting data points, do not reconcile them. Present both sides and label them **Conflicting Evidence**.

**Critical Assessment:** Build Strengths/Weaknesses from available data only. Be skeptical, not optimistic.

**Uncertainty Labeling:** If a claim comes from a non-definitive source, prefix it: "Unverified sources suggest..."

**Ambiguity Check:** If the objective or area of interest is still too broad after the interview, stop and ask for more clarity before generating the report.
