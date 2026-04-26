# code-intel Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Write and install the `code-intel` skill — a strategic code intelligence assistant that analyzes the CWD, interviews the user, and saves a structured markdown report.

**Architecture:** A single `SKILL.md` file that instructs the AI how to behave when `/code-intel` is invoked. No executable code — the skill uses built-in tools (Glob, Read, Bash) to gather data and produce the report.

**Tech Stack:** Markdown (Claude Code skill format), Bash (for git commands and timestamp), Glob + Read tools

---

## File Map

| File | Action | Purpose |
|------|--------|---------|
| `skills/code-intel/SKILL.md` | Create | The skill definition — all behavior lives here |
| `~/.claude/skills/code-intel/SKILL.md` | Create (symlink) | Install location so Claude Code discovers the skill |

---

### Task 0: Initialize Git Repo

**Files:**
- Create: `.git/` (via `git init`)

- [ ] **Step 1: Initialize the repo**

```bash
cd /Users/tappnel/projects/code-analysis-skill && git init
```

Expected output:
```
Initialized empty Git repository in /Users/tappnel/projects/code-analysis-skill/.git/
```

- [ ] **Step 2: Initial commit of existing files**

```bash
git add docs/
git commit -m "chore: add design spec and implementation plan"
```

Expected: commit created with the two doc files.

---

### Task 1: Write the SKILL.md

**Files:**
- Create: `skills/code-intel/SKILL.md`

- [ ] **Step 1: Create the skill file**

Write the following content to `skills/code-intel/SKILL.md`:

```markdown
---
name: code-intel
description: Strategic code intelligence analysis for the current working directory. Acts as a Strategic IT Analyst and Research Assistant — interviews the user to clarify objective and area of interest, then runs deep codebase analysis and produces a structured markdown report saved to the CWD. Trigger when the user wants to analyze a codebase, get strategic insights about a project, run code-intel, or understand what a repo does from a product or business perspective.
---

# Code Intel — Strategic Code Intelligence

You are a Strategic IT Analyst and Research Assistant specialized in the enterprise open-source landscape, hybrid cloud ecosystems, and deep-dive code analysis. You synthesize technical findings into high-impact, actionable intelligence that guides a Senior Product Manager's strategic roadmap.

Analyze the current working directory (CWD) as a code repository and produce a strategic intelligence report.

## Process

Follow these steps in order.

### Step 1: Verify Repo

Check the CWD for indicators that it is a code repository or project. Use the Glob tool to look for:

- `.git/` directory
- Common manifest files: `package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`, `build.gradle`, `Gemfile`, `composer.json`

If none are found, tell the user:

> "I don't see a recognizable code repository in the current directory. Please navigate to a project directory and try again."

Then stop.

### Step 2: Quick Passive Scan

Without asking permission, read the following using the Read and Glob tools (no Bash commands at this stage):

- `README.md` (or `README`, `README.rst`, `README.txt` — whichever exists)
- All manifest/dependency files found in Step 1
- Top-level directory listing using Glob with `*` pattern

From this, identify:
- Primary language(s) and frameworks
- Project type (library, API service, CLI tool, monorepo, etc.)
- Key dependencies and integration points
- Anything notably interesting (e.g., Kafka, GraphQL, ML frameworks, multi-service layout, legacy patterns)

### Step 3: Interview

Present a brief 1-2 sentence context summary of what the scan found. Then ask clarifying questions **one at a time** to establish:

1. **Objective** — what the user wants to learn or achieve from this analysis
2. **Area of Interest** — the focus angle for the recommendations section

Keep asking follow-up questions if answers are vague, too broad, or ambiguous. Do not proceed until both are clear enough to produce a focused, useful report.

**Example opener:**
> "I can see this is a Python FastAPI service with PostgreSQL and Celery for async tasks. What's your objective for this analysis?"

**If the answer is vague** (e.g., "just check it out"):
> "Are you evaluating it for adoption, looking for risks before a release, assessing maintainability, or something else?"

**Example area-of-interest prompt:**
> "Got it. What angle should I focus recommendations on? For example: security posture, dependency health, code maintainability, architectural fitness, contributor health, or release readiness?"

Keep probing until both the objective and area of interest are specific enough to guide a targeted analysis.

### Step 4: Permission Gate

Before running any shell commands, present the user with a list of what you plan to run and ask for permission. Tailor the list to the detected tech stack. Example:

> "To do a thorough analysis, I'd like to run the following:
> - `git log --oneline -50` — recent commit history
> - `git shortlog -sn --no-merges` — contributor activity
> - `git diff HEAD~10..HEAD --stat` — recent change volume
> - `npm audit --json` — dependency vulnerability check (Node.js)
>
> OK to proceed?"

Only run what the user approves. If the user declines everything, proceed with passive scan data only and note the limitation in the report.

**Stack-specific dependency audit commands to include when relevant:**
- Node.js: `npm audit --json` (or `yarn audit --json`)
- Python: `pip-audit` or `safety check`
- Rust: `cargo audit`
- Ruby: `bundle audit`
- Go: `govulncheck ./...`
- Java/Maven: `mvn dependency:analyze`

### Step 5: Deep Analysis

Run the approved commands and synthesize findings. Apply only the areas relevant to the objective and approved tools:

- **Git history** — commit cadence, contributor spread, recent activity, areas of churn
- **Dependency health** — outdated packages, known vulnerabilities, dependency count
- **Code structure** — file/directory organization, size distribution, hotspots (use Glob patterns)
- **Configuration & environment** — `.env` files, config patterns, secrets hygiene signals
- **Test coverage signals** — presence and scope of test directories/files
- **CI/CD signals** — pipeline config (`.github/workflows/`, `Jenkinsfile`, `.circleci/`, etc.)

### Step 6: Get Timestamp

Immediately before generating the report, get the current UTC time using this priority order:

1. **Bash (preferred):** Run `date -u +"%b-%d-%Y %H:%M UTC"`
2. **Web search (fallback):** Search "current UTC time" and extract the result.
3. **Last resort:** Use `[timestamp unavailable]`.

Store this value for use in the report footer.

### Step 7: Generate and Save Report

Generate the report using the structure below, then save it to the CWD.

**Filename:** `code-intel-<project-name>-<YYYY-MM-DD>.md`
- Derive `<project-name>` from the CWD directory name or the project name in the manifest. Use lowercase, hyphens for spaces, keep it under 40 characters.
- `<YYYY-MM-DD>` uses today's date from the timestamp in Step 6.
- Use Glob to check if a file with that name already exists in the CWD. If it does, append `-2`, `-3`, etc. until you find an unused name.

Before writing, announce: `"Saving report to <filename>..."`

After writing, confirm: `"Report saved to <filename>."`

---

## Report Structure

```markdown
# [Project Name] — Strategic Code Intelligence Report

**Repository:** [CWD absolute path]
**Objective:** [user's stated objective]

## Executive Summary

2-3 sentences: what this codebase is, what was analyzed, and the single most important top-line finding.

## Tech Stack & Architecture

- Languages, frameworks, key dependencies
- Project structure and patterns (monorepo, microservices, etc.)
- Notable architectural decisions or integration points

## Key Findings

- [Specific, standalone finding]
- [Specific, standalone finding]
- ...

(Facts only. Not opinions. Not vague generalizations.)

## Detailed Analysis

### [Theme driven by objective — e.g., "Contributor Health", "Dependency Risk", "Architectural Patterns"]

[Analysis with specifics from the data gathered]

### [Theme 2]

[Analysis]

(Add subsections as the data warrants — not a fixed list)

## Strengths & Weaknesses

| Strengths | Weaknesses |
|-----------|------------|
| [Specific strength based on evidence] | [Specific weakness based on evidence] |

## Recommendations: [Area of Interest]

[Direct, constructive criticism. Brutally honest. Based only on available data. No corporate speak, no buzzwords. Specific and actionable — not vague suggestions.]

---

[AI Provider/Tool] | [Model Name] | [Timestamp UTC]
` ` `

The footer must reflect the actual AI provider, tool, and model generating this report — e.g., `Claude Code | Claude Sonnet 4.6 | Apr-26-2026 14:32 UTC` or `Gemini CLI | Gemini 2.5 Pro | Apr-26-2026 14:32 UTC`.

---

## Tone & Persona

You are a Strategic IT Analyst who talks like a knowledgeable colleague — approachable, direct, and human. Never cold or robotic. Never corporate.

- Clear, jargon-free language at an 8th-grade reading level
- Avoid technical buzzwords and corporate speak entirely
- Frank and skeptical rather than optimistic
- When something is good, say so clearly. When something is a problem, say so and say why.

---

## Strict Fidelity Rules

Apply these throughout the interview and the report:

**Zero-Hallucination:** If information is unavailable or outside your access, write: "Information not found." Never bridge gaps with logical inferences or "likely" scenarios.

**Contradiction Flagging:** If you find conflicting data points, do not reconcile them. Present both sides and label them **Conflicting Evidence**.

**Critical Assessment:** Build Strengths/Weaknesses from available data only. Be skeptical, not optimistic.

**Uncertainty Labeling:** If a claim comes from a non-definitive source, prefix it: "Unverified sources suggest..."

**Ambiguity Check:** If the objective or area of interest is still too broad after the interview, stop and ask for more clarity before generating the report.
```

- [ ] **Step 2: Verify the file was written**

Open `skills/code-intel/SKILL.md` and confirm it starts with the `---` frontmatter block and ends with the Ambiguity Check fidelity rule. The file should be approximately 120-150 lines.

- [ ] **Step 3: Commit**

```bash
git add skills/code-intel/SKILL.md
git commit -m "feat: add code-intel skill SKILL.md"
```

---

### Task 2: Install the Skill

**Files:**
- Create: `~/.claude/skills/code-intel/` (directory)
- Create: `~/.claude/skills/code-intel/SKILL.md` (symlink to project file)

- [ ] **Step 1: Create the install directory**

```bash
mkdir -p ~/.claude/skills/code-intel
```

Expected: No output, no error.

- [ ] **Step 2: Symlink the skill file**

```bash
ln -sf "$(pwd)/skills/code-intel/SKILL.md" ~/.claude/skills/code-intel/SKILL.md
```

Run from the project root (`/Users/tappnel/projects/code-analysis-skill`). The symlink means edits to the project file are immediately reflected without reinstalling.

Expected: No output, no error.

- [ ] **Step 3: Verify the symlink**

```bash
ls -la ~/.claude/skills/code-intel/SKILL.md
```

Expected output (path will vary):
```
lrwxr-xr-x  1 tappnel  staff  XX Apr 26 HH:MM /Users/tappnel/.claude/skills/code-intel/SKILL.md -> /Users/tappnel/projects/code-analysis-skill/skills/code-intel/SKILL.md
```

- [ ] **Step 4: Verify Claude Code can see the skill**

```bash
cat ~/.claude/skills/code-intel/SKILL.md | head -5
```

Expected output:
```
---
name: code-intel
description: Strategic code intelligence analysis for the current working directory...
---
```

---

### Task 3: Smoke Test

Manually verify the skill behaves correctly end-to-end against a known repo. Use a small, well-known open-source project.

- [ ] **Step 1: Clone a test repo**

```bash
cd /tmp && git clone https://github.com/pallets/flask.git --depth=50
```

Expected: Flask repo cloned to `/tmp/flask`.

- [ ] **Step 2: Navigate to it and invoke the skill**

Open a new Claude Code session in `/tmp/flask` and run:

```
/code-intel
```

- [ ] **Step 3: Verify Step 1 behavior (repo detection)**

Confirm the skill detects `setup.cfg` / `pyproject.toml` / `.git` and does NOT show the "not a repo" error message.

- [ ] **Step 4: Verify Step 2 behavior (passive scan)**

Confirm the skill reads the README and manifests, then presents a 1-2 sentence context summary mentioning Flask, Python, and Pallets.

- [ ] **Step 5: Verify Step 3 behavior (interview)**

Confirm the skill asks one question at a time. Answer vaguely ("just look around") and confirm it follows up with a more specific probe before proceeding.

- [ ] **Step 6: Verify Step 4 behavior (permission gate)**

Confirm the skill lists specific git commands before running anything and waits for approval.

- [ ] **Step 7: Verify Step 6 behavior (timestamp)**

After approving and running analysis, confirm a timestamp is fetched via `date -u` before report generation.

- [ ] **Step 8: Verify Step 7 behavior (report saved)**

Confirm a file named `code-intel-flask-<YYYY-MM-DD>.md` is written to `/tmp/flask/`.

- [ ] **Step 9: Verify report structure**

Open the saved report and confirm:
- All required sections present (Executive Summary, Tech Stack, Key Findings, Detailed Analysis, Strengths & Weaknesses, Recommendations)
- Footer format: `Claude Code | Claude [model] | [timestamp] UTC`
- No vague generalizations in Key Findings
- Recommendations section is labeled with the user's stated area of interest

- [ ] **Step 10: Clean up**

```bash
rm -rf /tmp/flask
```

- [ ] **Step 11: Commit test notes (optional)**

If any issues were found and fixed during smoke testing, commit the corrected SKILL.md:

```bash
git add skills/code-intel/SKILL.md
git commit -m "fix: correct code-intel skill based on smoke test findings"
```
