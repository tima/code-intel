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
- Git repository or recognizable project structure (package.json, requirements.txt, go.mod, Cargo.toml, etc.)
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
