# Test Scenario: Emergency Analysis Request

## Anti-Pattern Detection Test

This scenario verifies that code-intel correctly identifies when analysis is requested under emergency conditions.

## Triggering Context

User statement contains urgency indicators:
- "Production is down"
- "Need to decide in the next [< 30 minutes]"
- "Incident response"
- "Security breach"
- "Customer-facing outage"
- "Revenue-critical deadline"

**Example user requests:**
- "We have a production outage and need to decide whether to rollback this library. /code-intel"
- "Security incident - need quick assessment of this codebase now"
- "Deploy decision needed in 15 minutes, run code-intel"

## Expected Anti-Pattern Trigger

Code-intel should detect **Emergency Decision** pattern:
- Time pressure incompatible with thorough analysis
- Code-intel requires 5-15 minutes minimum (passive scan + interview + deep analysis + report generation)
- Emergency decisions need speed, not deliberation

## Expected Behavior

**STOP with message:**
```
Emergency detected. Code-intel analysis takes 5-15 minutes minimum (interview + deep analysis + report generation).

Emergency decisions need:
- Speed over thoroughness
- Rapid risk assessment, not comprehensive intelligence
- Reversible short-term actions + post-incident deep-dive

For immediate needs:
- Quick scan: Read README + recent commits manually
- Dependency check: Run `npm audit` or equivalent directly
- Git history: `git log --oneline -20` for recent activity

Once the incident is resolved, run code-intel for a thorough post-incident analysis.

Proceed anyway? Use --force to override (not recommended).
```

## User Override

If user provides `--force`, proceed but include caveat:
> **Note:** This report was generated under time pressure. For emergency decisions, consider this preliminary analysis that should be validated in a post-incident review.

## Why This Matters

Prevents slowing down incident response with a 10-minute interview + analysis process when the user needs answers in 60 seconds.
