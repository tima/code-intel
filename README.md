# code-intel

Deep strategic analysis of codebases — understand what you're looking at before you commit to it.

## What It Does

Acts as a Strategic IT Analyst who interviews you about your objective, runs deep codebase analysis, and produces structured markdown intelligence reports saved to your working directory.

Reports prioritize clarity over jargon with brutally honest assessments - no corporate speak, no buzzwords, just findings and actionable recommendations.

## Installation

```bash
git clone https://github.com/username/code-intel-skill.git ~/projects/code-intel-skill

ln -s ~/projects/code-intel-skill ~/.claude/skills/code-intel
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

### Advanced Usage

**Override Anti-Pattern Detection:**

Code-intel includes anti-pattern detection that stops analysis for trivial repositories, emergency decisions, and validation-seeking requests. Override with:

```
/code-intel --force
```

**When to use --force:**
- You genuinely need strategic analysis of a small/trivial repository
- You understand the time investment despite emergency context
- You want descriptive analysis despite having a predetermined decision

**Note:** Overridden anti-patterns will be noted as limitations in the generated report.

## Example Output

See [example Flask library analysis](examples/example-flask-library.md) for a sample report.

## Requirements

- Git repository or recognizable project structure (package.json, requirements.txt, go.mod, Cargo.toml, etc.)
- Optional: Dependency audit tools for your stack (npm audit, pip-audit, cargo audit, etc.)

## Documentation

- [Design Doc](docs/design.md) - Architecture, design decisions, and technical details
- [Examples](examples/) - Sample reports

## License

MIT
