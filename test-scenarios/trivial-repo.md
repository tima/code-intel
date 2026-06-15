# Test Scenario: Trivial Repository

## Anti-Pattern Detection Test

This scenario verifies that code-intel correctly identifies trivial repositories where analysis would provide minimal value.

## Repository Characteristics

**Passive scan detects:**
- Single-file project (e.g., `hello.py` or `README.md` + `script.sh`)
- No dependencies in manifest (empty `requirements.txt` or minimal `package.json` with zero dependencies)
- File count < 10
- No `.git/` directory OR commit count < 5
- Total lines of code < 200 (estimate from file sizes)

## Expected Anti-Pattern Trigger

Code-intel should detect **Trivial Repository** pattern:
- Too small for meaningful strategic analysis
- No architecture to evaluate
- No dependency health to assess
- No contributor patterns to analyze

## Expected Behavior

**STOP with message:**
```
This appears to be a trivial repository (< 10 files, minimal dependencies, < 5 commits).

Strategic code intelligence analysis is designed for substantial codebases where:
- Architecture decisions have been made
- Dependencies and integrations exist
- Multiple contributors or commit history exists
- There's meaningful structure to evaluate

For this repository, a manual code review would be more appropriate than a strategic intelligence report.

Skip code-intel? Use --force to override.
```

## User Override

If user provides `--force` flag, proceed with caveat in report:
```
**Analysis Limitations:** This is a trivial repository with minimal structure. Findings are constrained by limited available data.
```

## Why This Matters

Prevents wasting 5-10 minutes generating a report that says "this is a simple script with no meaningful architecture to evaluate."
