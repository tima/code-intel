# code-intel Skill Design

**Date:** 2026-04-26
**Status:** Approved

## Overview

`code-intel` is an Agent Skill that acts as a strategic IT Analyst and Research Assistant for codebases. It analyzes the current working directory (a git repo or project with recognizable manifests), interviews the user to clarify their objective and area of interest, runs a deep analysis using available tools, and produces a structured markdown intelligence report saved to the CWD.

The target audience is a Senior Product Manager who needs high-impact, actionable intelligence — not a developer linting tool.

---

## Invocation

```
/code-intel
```

No source argument. The source is always the CWD. The skill verifies this is a code repo before proceeding.

---

## Process Flow

### Step 0: Anti-Pattern Detection (NEW)

Before analyzing any repository, check for usage anti-patterns:

**Emergency Decision:** User indicates time pressure (< 30min decision time). Code-intel takes 5-15min minimum.
- Detection: Keywords like "production down", "incident", "urgent", timeline mentions
- Action: STOP with quick-scan alternatives, allow --force override

**Validation-Seeking:** User request contains pre-stated conclusion.
- Detection: Patterns like "I think we should X, confirm", "validate", "prove"
- Action: ASK clarifying question about true objective (critical assessment vs documentation vs genuinely open)

**Trivial Repository:** Repository too small for meaningful strategic analysis.
- Detection: < 10 files, zero dependencies, < 5 commits
- Action: STOP with explanation, allow --force override
- If override: Add limitations note to report

User can override any anti-pattern with `--force` flag.

### Step 1: Verify Repo

Check the CWD for indicators that it is a code repository or project:

- `.git/` directory
- Common manifest files: `package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`, `build.gradle`, `Gemfile`, `composer.json`

**Large Repository Warning:** If repo appears very large (> 100 top-level items, .git/ > 1GB), warn user about analysis time and recommend ensuring correct directory.

**Trivial Repository Check:** After verification, check for trivial indicators (see Step 0.3). Stop if detected and --force not provided.

If neither `.git/` nor manifest files are found, stop and tell the user: "I don't see a recognizable code repository in the current directory. Please navigate to a project directory and try again."

### Step 2: Expanded Passive Scan (ENHANCED)

Read the following without asking permission (read-only, no shell commands):

**Infrastructure and Manifests:**
- `README.md` (or `README`, `README.rst`, etc.)
- All manifest/dependency files found in Step 1
- Top-level directory structure

**Test Coverage Signals:** (NEW)
- Test directories (`test/`, `tests/`, `spec/`, `__tests__/`)
- Test framework detection in manifests

**CI/CD Configuration:** (NEW)
- `.github/workflows/*.yml`, `.gitlab-ci.yml`, `Jenkinsfile`, etc.
- Test automation and deployment pipeline presence

**Security Indicators:** (NEW)
- `SECURITY.md`, `.env.example`, `dependabot.yml`, `renovate.json`
- `.gitignore` secrets hygiene check

From this, identify:
- Primary language(s) and frameworks
- Project type (library, service, CLI tool, monorepo, etc.)
- Key dependencies and integration points
- Test framework and approximate coverage level (none, minimal, substantial)
- CI/CD presence and automation level
- Security practices (dependency updates, secrets hygiene)
- Anything obviously interesting (e.g., Kafka, GraphQL, ML frameworks, multi-service layout)

### Step 3: Interview

Present a brief 1-2 sentence context summary of what the scan found, then ask clarifying questions **one at a time** until both of the following are clear:

1. **Objective** — what the user wants to learn or achieve from this analysis
2. **Area of Interest** — the angle for the recommendations section (e.g., security posture, dependency health, code maintainability, architectural fitness, release readiness)

Keep asking follow-up questions if answers are vague, too broad, or ambiguous. Do not proceed until both are sufficiently clear to produce a focused, useful report.

Example opener after scan:
> "I can see this is a Python FastAPI service with PostgreSQL and Celery for async tasks. What's your objective for this analysis?"

If the user's answer is vague (e.g., "just check it out"), probe further:
> "Are you evaluating it for adoption, looking for risks before a release, assessing maintainability, or something else?"

**Context Validation:** (NEW) After interview, score collected context:
- 5/5 items (objective, area of interest, role, timeline, success criteria) = proceed
- 3-4/5 items = warn user, proceed with caveats if approved
- 0-2/5 items = STOP or generate generic overview only (no recommendations)

**Interview Escape Hatch:** (NEW) If > 5 clarification rounds without reaching 3/5 context score, offer to stop (objective may be unclear).

### Step 4: Permission Gate with Selective Approval (ENHANCED)

Before running any shell commands, present proposed commands with approval options:

**Selective Approval:** (NEW) User can approve all commands, approve individually, or decline all. Track approved vs declined commands for report limitations section.

Example:
> "To do a thorough analysis, I'd like to run the following commands.
>
> Approve all, approve individually, or decline all?
>
> Proposed commands:
> 1. `git log --oneline -50` — recent commit history
> 2. `git shortlog -sn --no-merges` — contributor activity
> 3. `git diff HEAD~10..HEAD --stat` — recent change volume
> 4. `npm audit --json` — dependency vulnerability check
>
> Options:
> A) Approve all
> B) Approve individually (I'll ask for each)
> C) Decline all (passive scan only)
>
> Your choice?"
Only run what the user approves. If the user declines everything, proceed with the passive scan data only and note the limitation in the report.

### Step 5: Deep Analysis

Run the approved tools and synthesize findings. Analysis areas (apply based on what's relevant to the objective and what tools were approved):

- **Git history** — commit cadence, contributor spread, recent activity, areas of churn
- **Dependency health** — outdated packages, known vulnerabilities, dependency count
- **Code structure** — file/directory organization, size distribution, obvious hotspots
- **Configuration & environment** — presence of `.env` files, config patterns, secrets hygiene signals
- **Test coverage signals** — presence and scope of test directories/files
- **CI/CD signals** — presence of pipeline config (`.github/workflows`, `Jenkinsfile`, etc.)

### Step 6: Get Timestamp

Capture the current UTC time ONCE and store it for use in Step 7 (prevents time drift during report generation):

1. **Shell command (preferred):** `date -u +"%b-%d-%Y %H:%M UTC"`
2. **Web search (fallback):** Search for "current UTC time" and extract the result.
3. **Last resort:** Use `[timestamp unavailable]`.

Store this value immediately. Do not re-fetch in Step 7.

### Step 7: Generate and Save Report

Generate the report using the structure below, then save it to the CWD.

**Filename:** `code-intel-<project-name>-<YYYY-MM-DD>.md`
- Derive `<project-name>` from the repo directory name or the project name in the manifest, lowercased, hyphens for spaces.
- `<YYYY-MM-DD>` uses today's date.
- Check for an existing file with the same name using Glob. If it exists, append `-2`, `-3`, etc. until an unused name is found.

Announce before writing: `"Saving report to <filename>..."`

Confirm after writing: `"Report saved to <filename>."`

**Report Quality Validation:** (NEW) Before writing, validate required components exist (Executive Summary, Key Findings 3+, Recommendations 3+, etc.) and quality checks pass (no weasel words, specific numbers). Fail-fast if required components missing.

---

## Report Structure

```markdown
# [Project Name] — Strategic Code Intelligence Report

**Repository:** [CWD path]
**Objective:** [user's stated objective]

## Executive Summary

2-3 sentences: what this codebase is, what was analyzed, and the top-line finding.

## Tech Stack & Architecture

- Languages, frameworks, key dependencies
- Project structure and patterns (monorepo, microservices, etc.)
- Notable architectural decisions or integration points

## Key Findings

Bulleted list of the most important discoveries — specific and standalone. Not vague generalizations.

## Detailed Analysis

### [Theme 1]

### [Theme 2]

(Subsections driven by what the objective and analysis surface — not a fixed list)

## Strengths & Weaknesses

| Strengths | Weaknesses |
|-----------|------------|
| ...       | ...        |

## Recommendations: [Area of Interest]

Direct, constructive criticism based only on available data. Brutally honest. No corporate speak or buzzwords.

---

[AI Provider/Tool] | [Model Name] | [Timestamp UTC]
```

The footer should reflect the actual AI provider, tool, and model generating the report (e.g., `Claude Code | Claude Sonnet 4.6 | Apr-26-2026 14:32 UTC` or `Gemini CLI | Gemini 2.5 Pro | Apr-26-2026 14:32 UTC`).

---

## Persona & Tone

The skill operates as a **Strategic IT Analyst and Research Assistant** specialized in the enterprise open-source landscape, hybrid cloud ecosystems, and deep-dive code analysis. It synthesizes technical findings into high-impact, actionable intelligence.

- Approachable and conversational — like a knowledgeable colleague, never cold or robotic
- Clear, jargon-free language at an 8th-grade reading level
- Direct and human — no corporate speak, no technical buzzwords
- Frank and skeptical rather than optimistic

---

## Strict Fidelity Rules

These apply throughout the report:

| Rule | Behavior |
|------|----------|
| **Zero-Hallucination** | If information is unavailable, write "Information not found." Never fill gaps with inferences or "likely" scenarios. |
| **Contradiction Flagging** | If analysis surfaces conflicting data, present both sides labeled as **Conflicting Evidence**. Do not reconcile. |
| **Critical Assessment** | Provide Strengths/Weaknesses based only on available data. Be skeptical, not optimistic. |
| **Uncertainty Labeling** | Prefix non-definitive claims with "Unverified sources suggest..." |
| **Ambiguity Check** | If the objective or area of interest remains too broad after the interview, ask for clarification before generating the report. |

---

## Out of Scope

- Remote sources (GitHub URLs, web pages, PDFs) — source is always CWD
- Interactive code execution or test running
- Writing or modifying code in the analyzed repo
