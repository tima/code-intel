# Test Scenario: Validation-Seeking Analysis Request

## Anti-Pattern Detection Test

This scenario verifies that code-intel correctly identifies when the user is seeking validation of an existing decision rather than open exploration.

## Triggering Context

User request contains decided position:
- "I think we should migrate to [X], can you analyze this codebase to confirm?"
- "We've decided to adopt this library, run code-intel to validate"
- "Management wants us to use this repo, need to prove it's a good fit"
- "I'm pretty sure this codebase is [quality judgment], can you verify?"

**Key indicators:**
- Question contains pre-stated conclusion
- Asks for "confirmation" or "validation" rather than "analysis"
- Political pressure mentioned
- Sunk cost already incurred (time/money invested)

## Expected Anti-Pattern Trigger

Code-intel should detect **Validation-Seeking** pattern:
- Analysis works best for open questions, not confirmation bias
- If decision is already made, what's the actual objective?
- Risk of cherry-picking findings to support predetermined outcome

## Expected Behavior

**ASK CLARIFYING QUESTION:**
```
It sounds like you've already leaned toward [conclusion from user statement]. 

Code-intel works best for open exploration - analyzing what IS, not confirming what you hope to find.

Are you:
A) Looking for weaknesses/risks you might have missed (critical assessment)?
B) Genuinely open to a "don't adopt this" conclusion if evidence warrants?
C) Seeking documentation of existing decision for stakeholders?

If C, I can generate a report, but I'll frame it as descriptive (what the codebase is) rather than prescriptive (whether to adopt it).
```

## User Clarification Paths

**If user chooses A (critical assessment):**
Proceed with objective: "Identify risks and weaknesses before adoption decision"

**If user chooses B (truly open):**
Proceed with objective: "Evaluate whether this codebase fits [stated need]"

**If user chooses C (documentation):**
Proceed but set objective: "Document existing codebase characteristics for stakeholder review"
Report should be descriptive, not prescriptive (no "should adopt" or "should avoid" language)

## Why This Matters

Prevents generating reports that become confirmation bias tools. Forces user to clarify actual objective rather than seeking validation.
