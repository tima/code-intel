# code-intel

Analyze codebases to understand how they work and how to extend/integrate them.

## What It Does

Acts as a code analyst who interviews you about your specific question, traces relevant code paths, and produces a structured markdown report explaining architecture, entry points, extension points, and integration seams.

Reports are laser-focused on answering your tactical code question — not project health or sustainability.

## When to Use

**Good fit:**
- "How would we add authentication?"
- "How does the payment flow work?"
- "Where would we integrate Kafka?"
- "How do we extend this with plugins?"
- "What happens when X fails?"

**When NOT to use:**
- You need to fix a specific bug (use debugging, not analysis)
- You need a quick security audit (use tool-specific audits: `npm audit`, `cargo audit`)
- You're evaluating project health/team sustainability (no longer what code-intel does)

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
1. Verify you're in a code repository
2. Scan code structure (entry points, abstractions, extension mechanisms)
3. Ask what you want to understand about the code
4. Trace relevant code paths
5. Generate and save a report

Reports are saved as `code-intel-<project-name>-<YYYY-MM-DD>.md`.

## Example Output

See [example Django integration analysis](examples/example-django-integration.md) for a sample report structure.

## Requirements

- Git repository or recognizable project structure (package.json, requirements.txt, go.mod, Cargo.toml, etc.)

## Documentation

- [Design Doc](docs/design.md) - Design rationale and architecture
- [Examples](examples/) - Sample reports

## License

MIT
