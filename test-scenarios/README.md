# Code-Intel Test Scenarios

Baseline scenarios for validating code-intel behavior across different repository types and edge cases.

## Purpose

These scenarios help verify that code-intel:
- Correctly identifies repository types
- Asks appropriate interview questions based on passive scan findings
- Generates relevant recommendations for different tech stacks
- Handles edge cases gracefully

## Scenario Structure

Each scenario file contains:
- **Repository context** - What the passive scan should detect
- **Expected interview questions** - What clarifications are appropriate
- **Expected report sections** - What themes should appear in analysis
- **Anti-pattern indicators** - When code-intel should NOT proceed

## Using These Scenarios

Before significant changes to SKILL.md:
1. Review each scenario
2. Manually test against a representative repository
3. Verify the skill still handles the scenario appropriately
4. Update scenarios if expected behavior changes intentionally

## Scenarios

- [Trivial Repository](trivial-repo.md) - Anti-pattern detection
- [Monorepo](monorepo.md) - Multi-language, multi-service
- [Emergency Request](emergency-request.md) - Anti-pattern: time pressure
- [Validation-Seeking](validation-seeking.md) - Anti-pattern: predetermined conclusion
