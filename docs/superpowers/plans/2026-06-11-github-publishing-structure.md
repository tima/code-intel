# GitHub Publishing Structure Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restructure code-intel-skill repo for GitHub publishing with flattened layout, quality documentation, and 2-command installation.

**Architecture:** Flatten skills/code-intel/SKILL.md to root, create user-facing README.md, consolidate design docs, add MIT license, create examples directory with sample reports. Remove internal tooling directories.

**Tech Stack:** Git, Markdown, Bash

---

## File Map

**Files to move:**
- `skills/code-intel/SKILL.md` → `SKILL.md`
- `docs/code-intel-flask-2026-04-27.md` → `examples/example-flask-library.md`

**Files to create:**
- `README.md` (new, user-facing)
- `LICENSE` (new, MIT)
- `docs/design.md` (consolidate from superpowers/specs)
- `examples/example-flask-library.md` (moved/renamed)

**Directories to remove:**
- `skills/` (entire directory)
- `.claude/` (entire directory)
- `docs/superpowers/` (entire directory)

**Final structure:**
```
code-intel-skill/
├── README.md
├── LICENSE
├── SKILL.md
├── examples/
│   └── example-flask-library.md
└── docs/
    └── design.md
```

---

### Task 1: Create README.md

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write README.md content**

```markdown
# code-intel

Strategic code intelligence analysis for Claude Code

## What It Does

Analyzes your codebase and generates strategic intelligence reports. Acts as a Strategic IT Analyst - interviews you to understand your objective, runs deep analysis, produces structured markdown reports saved to your working directory.

Reports are written at an 8th grade reading level with brutally honest assessments - no corporate speak, no buzzwords, just clear findings and actionable recommendations.

## Installation

```bash
# Clone the repository
git clone https://github.com/username/code-intel-skill.git

# Create symlink to Claude Code skills directory
ln -s /path/to/code-intel-skill ~/.claude/skills/code-intel

# Verify installation
claude skills list | grep code-intel
```

## Usage

Navigate to a code repository and invoke:

```
/code-intel
```

The skill will:

1. Verify you're in a code repository (checks for `.git/` or manifest files)
2. Run a quick passive scan of README and dependencies
3. Interview you about your objective and area of interest
4. Ask permission before running deeper analysis commands
5. Generate and save a report to your working directory

Reports are saved as `code-intel-<project-name>-<YYYY-MM-DD>.md`.

## Example Output

See [example Flask library analysis](examples/example-flask-library.md) for a sample report.

## Requirements

- Claude Code
- Git repository or recognizable project structure (package.json, requirements.txt, go.mod, etc.)
- Optional: Dependency audit tools for your stack (npm audit, pip-audit, cargo audit, etc.)

## Documentation

- [Design Doc](docs/design.md) - Architecture, design decisions, and technical details
- [Examples](examples/) - Sample reports

## Uninstall

```bash
# Remove symlink
rm ~/.claude/skills/code-intel

# Optionally remove cloned repo
rm -rf /path/to/code-intel-skill
```

## Updates

```bash
# Pull latest changes (automatically available via symlink)
cd /path/to/code-intel-skill
git pull
```

## License

MIT
```

- [ ] **Step 2: Verify file created**

Run: `ls -la README.md`
Expected: File exists at repo root

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: add README for GitHub publishing"
```

---

### Task 2: Create MIT License

**Files:**
- Create: `LICENSE`

- [ ] **Step 1: Write LICENSE file**

```text
MIT License

Copyright (c) 2026 code-intel-skill contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] **Step 2: Verify file created**

Run: `ls -la LICENSE`
Expected: File exists at repo root

- [ ] **Step 3: Commit**

```bash
git add LICENSE
git commit -m "chore: add MIT license"
```

---

### Task 3: Move SKILL.md to Root

**Files:**
- Move: `skills/code-intel/SKILL.md` → `SKILL.md`

- [ ] **Step 1: Move SKILL.md to root**

```bash
git mv skills/code-intel/SKILL.md SKILL.md
```

- [ ] **Step 2: Verify move**

Run: `ls -la SKILL.md && ls -la skills/code-intel/`
Expected: SKILL.md at root, skills/code-intel/ empty or directory doesn't exist

- [ ] **Step 3: Commit**

```bash
git commit -m "refactor: move SKILL.md to root for flattened structure"
```

---

### Task 4: Create Examples Directory and Move Flask Report

**Files:**
- Create: `examples/`
- Move: `docs/code-intel-flask-2026-04-27.md` → `examples/example-flask-library.md`

- [ ] **Step 1: Create examples directory**

```bash
mkdir -p examples
```

- [ ] **Step 2: Move and rename Flask report**

```bash
git mv docs/code-intel-flask-2026-04-27.md examples/example-flask-library.md
```

- [ ] **Step 3: Verify structure**

Run: `ls -la examples/`
Expected: Directory exists with example-flask-library.md inside

- [ ] **Step 4: Commit**

```bash
git add examples/
git commit -m "docs: create examples directory with Flask library analysis"
```

---

### Task 5: Consolidate Design Documentation

**Files:**
- Create: `docs/design.md`
- Source: `docs/superpowers/specs/2026-04-26-code-intel-design.md`

- [ ] **Step 1: Write consolidated design doc**

```markdown
# code-intel Design Documentation

## Overview

`code-intel` is a Claude Code skill that acts as a strategic IT Analyst and Research Assistant for codebases. It analyzes the current working directory (a git repo or project with recognizable manifests), interviews the user to clarify their objective and area of interest, runs a deep analysis using available tools, and produces a structured markdown intelligence report saved to the working directory.

The target audience is a Senior Product Manager who needs high-impact, actionable intelligence - not a developer linting tool.

## Invocation

```
/code-intel
```

No source argument. The source is always the current working directory. The skill verifies this is a code repo before proceeding.

## Process Flow

The skill follows these 7 steps in order:

### Step 1: Verify Repo

Check the current working directory for indicators that it is a code repository or project:

- `.git/` directory
- Common manifest files: `package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`, `build.gradle`, `Gemfile`, `composer.json`

If none are found, stop and tell the user: "I don't see a recognizable code repository in the current directory. Please navigate to a project directory and try again."

### Step 2: Quick Passive Scan

Read the following without asking permission (read-only, no shell commands):

- `README.md` (or `README`, `README.rst`, etc.)
- All manifest/dependency files found in Step 1
- Top-level directory structure (file listing only, no traversal)

From this, identify:
- Primary language(s) and frameworks
- Project type (library, service, CLI tool, monorepo, etc.)
- Key dependencies and integration points
- Anything obviously interesting (e.g., Kafka, GraphQL, ML frameworks, multi-service layout)

### Step 3: Interview

Present a brief 1-2 sentence context summary of what the scan found, then ask clarifying questions one at a time until both of the following are clear:

1. **Objective** - what the user wants to learn or achieve from this analysis
2. **Area of Interest** - the angle for the recommendations section (e.g., security posture, dependency health, code maintainability, architectural fitness, release readiness)

Keep asking follow-up questions if answers are vague, too broad, or ambiguous. Do not proceed until both are sufficiently clear to produce a focused, useful report.

Example opener after scan:
> "I can see this is a Python FastAPI service with PostgreSQL and Celery for async tasks. What's your objective for this analysis?"

If the user's answer is vague (e.g., "just check it out"), probe further:
> "Are you evaluating it for adoption, looking for risks before a release, assessing maintainability, or something else?"

### Step 4: Permission Gate

Before running any shell commands or deeper analysis tools, present the user with a list of what the skill plans to run and ask for permission. Example:

> "To do a thorough analysis, I'd like to run the following:
> - `git log --oneline -50` - recent commit history
> - `git shortlog -sn --no-merges` - contributor activity
> - `git diff HEAD~10..HEAD --stat` - recent change volume
> - [dependency audit command relevant to the stack]
>
> OK to proceed?"

Only run what the user approves. If the user declines everything, proceed with the passive scan data only and note the limitation in the report.

Stack-specific dependency audit commands to include when relevant:

| Stack | Command |
|---|---|
| Node.js | `npm audit --json` or `yarn audit --json` |
| Python | `pip-audit` or `safety check` |
| Rust | `cargo audit` |
| Ruby | `bundle audit` |
| Go | `govulncheck ./...` |
| Java/Maven | `mvn dependency:analyze` |

### Step 5: Deep Analysis

Run the approved tools and synthesize findings. Analysis areas (apply based on what's relevant to the objective and what tools were approved):

- **Git history** - commit cadence, contributor spread, recent activity, areas of churn
- **Dependency health** - outdated packages, known vulnerabilities, dependency count
- **Code structure** - file/directory organization, size distribution, obvious hotspots
- **Configuration and environment** - presence of `.env` files, config patterns, secrets hygiene signals
- **Test coverage signals** - presence and scope of test directories/files
- **CI/CD signals** - presence of pipeline config (`.github/workflows`, `Jenkinsfile`, etc.)

### Step 6: Get Timestamp

Immediately before generating the report, get the current UTC time:

1. **Bash (preferred):** `date -u +"%b-%d-%Y %H:%M UTC"`
2. **Web search (fallback):** Search for "current UTC time" and extract the result
3. **Last resort:** Use `[timestamp unavailable]`

Store this value for use in the report footer.

### Step 7: Generate and Save Report

Generate the report using the structure below, then save it to the current working directory.

**Filename:** `code-intel-<project-name>-<YYYY-MM-DD>.md`
- Derive `<project-name>` from the repo directory name or the project name in the manifest, lowercased, hyphens for spaces
- `<YYYY-MM-DD>` uses today's date
- Check for an existing file with the same name. If it exists, append `-2`, `-3`, etc. until an unused name is found

Announce before writing: `"Saving report to <filename>..."`

Confirm after writing: `"Report saved to <filename>."`

## Report Structure

```markdown
# [Project Name] - Strategic Code Intelligence Report

**Repository:** [absolute path to working directory]
**Objective:** [user's stated objective]

## Executive Summary

2-3 sentences: what this codebase is, what was analyzed, and the top-line finding.

## Tech Stack and Architecture

- Languages, frameworks, key dependencies
- Project structure and patterns (monorepo, microservices, etc.)
- Notable architectural decisions or integration points

## Key Findings

Bulleted list of the most important discoveries - specific and standalone. Not vague generalizations.

## Detailed Analysis

### [Theme 1]

### [Theme 2]

(Subsections driven by what the objective and analysis surface - not a fixed list)

## Strengths and Weaknesses

| Strengths | Weaknesses |
|-----------|------------|
| ...       | ...        |

## Recommendations: [Area of Interest]

Direct, constructive criticism based only on available data. Brutally honest. No corporate speak or buzzwords.

---

[AI Provider/Tool] | [Model Name] | [Timestamp UTC]
```

The footer should reflect the actual AI provider, tool, and model generating the report (e.g., `Claude Code | Claude Sonnet 4.6 | Apr-26-2026 14:32 UTC`).

## Persona and Tone

The skill operates as a **Strategic IT Analyst and Research Assistant** specialized in the enterprise open-source landscape, hybrid cloud ecosystems, and deep-dive code analysis. It synthesizes technical findings into high-impact, actionable intelligence.

- Approachable and conversational - like a knowledgeable colleague, never cold or robotic
- Clear, jargon-free language at an 8th grade reading level
- Direct and human - no corporate speak, no technical buzzwords
- Frank and skeptical rather than optimistic

## Strict Fidelity Rules

These apply throughout the report:

| Rule | Behavior |
|------|----------|
| **Zero-Hallucination** | If information is unavailable, write "Information not found." Never fill gaps with inferences or "likely" scenarios. |
| **Contradiction Flagging** | If analysis surfaces conflicting data, present both sides labeled as **Conflicting Evidence**. Do not reconcile. |
| **Critical Assessment** | Provide Strengths/Weaknesses based only on available data. Be skeptical, not optimistic. |
| **Uncertainty Labeling** | Prefix non-definitive claims with "Unverified sources suggest..." |
| **Ambiguity Check** | If the objective or area of interest remains too broad after the interview, ask for clarification before generating the report. |

## Out of Scope

- Remote sources (GitHub URLs, web pages, PDFs) - source is always current working directory
- Interactive code execution or test running
- Writing or modifying code in the analyzed repo

## Design Decisions

**Why 7 steps?** Breaking the process into discrete stages makes the workflow transparent and allows users to understand what's happening at each phase.

**Why the permission gate?** Running shell commands without consent is intrusive. The permission gate respects user agency and allows them to opt out of commands they don't trust or need.

**Why passive scan first?** Reading files before asking questions gives context for better interview questions. It also means the skill can provide value even if the user declines all shell commands.

**Why interview before analysis?** Generic reports are useless. The interview ensures the report is tailored to the user's actual needs and provides relevant recommendations.

**Why strict fidelity rules?** Generic AI reports are full of hallucinations and safe generalizations. The fidelity rules force the skill to only report what it actually found, making the output trustworthy.
```

- [ ] **Step 2: Verify file created**

Run: `ls -la docs/design.md`
Expected: File exists in docs/ directory

- [ ] **Step 3: Commit**

```bash
git add docs/design.md
git commit -m "docs: consolidate design documentation"
```

---

### Task 6: Remove Internal Directories

**Files:**
- Remove: `skills/` directory
- Remove: `.claude/` directory
- Remove: `docs/superpowers/` directory

- [ ] **Step 1: Remove skills directory**

```bash
git rm -r skills/
```

- [ ] **Step 2: Remove .claude directory**

```bash
git rm -r .claude/
```

- [ ] **Step 3: Remove docs/superpowers directory**

```bash
git rm -r docs/superpowers/
```

- [ ] **Step 4: Verify removal**

Run: `ls -la`
Expected: No skills/, .claude/, or docs/superpowers/ directories

- [ ] **Step 5: Commit**

```bash
git commit -m "refactor: remove internal tooling directories for GitHub publishing"
```

---

### Task 7: Verify Final Structure

**Files:**
- Verify: All files in correct locations

- [ ] **Step 1: List root directory**

Run: `ls -la`
Expected output should include:
```
README.md
LICENSE
SKILL.md
examples/
docs/
.git/
```

- [ ] **Step 2: List examples directory**

Run: `ls -la examples/`
Expected: `example-flask-library.md`

- [ ] **Step 3: List docs directory**

Run: `ls -la docs/`
Expected: `design.md`

- [ ] **Step 4: Verify no internal directories remain**

Run: `ls -la | grep -E "(skills|\.claude)"`
Expected: No output (directories removed)

- [ ] **Step 5: Check git status**

Run: `git status`
Expected: Working tree clean, all changes committed

---

### Task 8: Final Commit and Tag

**Files:**
- None (git operations only)

- [ ] **Step 1: Review git log**

Run: `git log --oneline -10`
Expected: See all restructuring commits

- [ ] **Step 2: Create annotated tag for v1.0.0**

```bash
git tag -a v1.0.0 -m "Release v1.0.0: GitHub publishing structure

- Flattened structure with SKILL.md at root
- User-facing README.md with installation instructions
- MIT license
- Examples directory with Flask library analysis
- Consolidated design documentation
- Removed internal tooling directories"
```

- [ ] **Step 3: Verify tag created**

Run: `git tag -l`
Expected: `v1.0.0` in tag list

- [ ] **Step 4: Show tag details**

Run: `git show v1.0.0`
Expected: Tag annotation and commit details displayed

---

## Success Criteria

After completing all tasks:

1. Repository has flattened structure with SKILL.md at root
2. README.md provides clear installation and usage instructions
3. MIT LICENSE file exists
4. examples/ directory contains sample Flask library report
5. docs/design.md consolidates design documentation
6. No internal directories (skills/, .claude/, docs/superpowers/) remain
7. All changes committed with clear messages
8. v1.0.0 tag created for initial public release
9. Repository ready for `git remote add origin` and `git push`
