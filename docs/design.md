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
