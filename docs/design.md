# code-intel Skill Design

**Date:** 2026-04-26
**Status:** Approved

## Overview

`code-intel` is a Claude Code skill that acts as a strategic IT Analyst and Research Assistant for codebases. It analyzes the current working directory (a git repo or project with recognizable manifests), interviews the user to clarify their objective and area of interest, runs a deep analysis using available tools, and produces a structured markdown intelligence report saved to the CWD.

The target audience is a Senior Product Manager who needs high-impact, actionable intelligence — not a developer linting tool.

---

## Invocation

```
/code-intel
```

No source argument. The source is always the CWD. The skill verifies this is a code repo before proceeding.

---

## Process Flow

### Step 1: Verify Repo

Check the CWD for indicators that it is a code repository or project:

- `.git/` directory
- Common manifest files: `package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`, `build.gradle`, `Gemfile`, `composer.json`

If none are found, stop and tell the user: "I don't see a recognizable code repository in the current directory. Please navigate to a project directory and try again."

### Step 2: Quick Passive Scan

Read the following without asking permission (read-only, no shell commands):

- `README.md` (or `README`, `README.rst`, etc.)
- All manifest/dependency files found in Step 1
- Top-level directory structure (`ls` equivalent — file listing only, no traversal)

From this, identify:
- Primary language(s) and frameworks
- Project type (library, service, CLI tool, monorepo, etc.)
- Key dependencies and integration points
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

### Step 4: Permission Gate

Before running any shell commands or deeper analysis tools, present the user with a list of what the skill plans to run and ask for permission. Example:

> "To do a thorough analysis, I'd like to run the following:
> - `git log --oneline -50` — recent commit history
> - `git shortlog -sn --no-merges` — contributor activity
> - `git diff HEAD~10..HEAD --stat` — recent change volume
> - [dependency audit command relevant to the stack]
>
> OK to proceed?"

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

Immediately before generating the report, get the current UTC time:

1. **Bash (preferred):** `date -u +"%b-%d-%Y %H:%M UTC"`
2. **Web search (fallback):** Search for "current UTC time" and extract the result.
3. **Last resort:** Use `[timestamp unavailable]`.

Store this value for use in the report footer.

### Step 7: Generate and Save Report

Generate the report using the structure below, then save it to the CWD.

**Filename:** `code-intel-<project-name>-<YYYY-MM-DD>.md`
- Derive `<project-name>` from the repo directory name or the project name in the manifest, lowercased, hyphens for spaces.
- `<YYYY-MM-DD>` uses today's date.
- Check for an existing file with the same name using Glob. If it exists, append `-2`, `-3`, etc. until an unused name is found.

Announce before writing: `"Saving report to <filename>..."`

Confirm after writing: `"Report saved to <filename>."`

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

The footer should reflect the actual AI provider, tool, and model generating the report (e.g., `Claude Code | Claude Sonnet 4.6 | Apr-26-2026 14:32 GMT` or `Gemini CLI | Gemini 2.5 Pro | Apr-26-2026 14:32 GMT`).

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
