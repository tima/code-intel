# GitHub Publishing Structure Design

**Date:** 2026-06-11
**Status:** Approved

## Overview

Restructure the code-intel-skill repository for public GitHub publishing, optimizing for ease of discovery, first-time setup, and documentation quality. The structure follows the counsel-this pattern (single skill, flattened layout) while maintaining design documentation quality standards.

## Goals

**Primary:**
1. Ease of discovery and first-time setup
2. Documentation quality

**Secondary:**
- Match established patterns from counsel-this and need-to-know repositories
- Enable users to clone and symlink in 2 commands
- Provide clear examples of output quality
- Support git-pull updates via symlink

## Target Structure

```
code-intel-skill/
├── README.md
├── LICENSE
├── SKILL.md
├── examples/
│   └── [sample-reports]
└── docs/
    └── design.md
```

### Structure Rationale

**Flattened root layout:** Matches counsel-this pattern for single-skill repositories. SKILL.md at root means users can symlink the entire repo directory to `~/.claude/skills/code-intel`.

**No skills/ wrapper:** Since this is a single-skill repo (not multi-skill like need-to-know), the extra nesting adds no value. Simpler is better for discovery.

**No .github/ folder:** Keep minimal - no CI, issue templates, or GitHub-specific tooling at launch.

**examples/ at root:** Easy to find, demonstrates output quality immediately.

**docs/ for technical details:** Design documentation and contributor information separate from user-facing README.

## Installation Flow

### Prerequisites
- Claude Code installed
- Git installed
- Basic terminal familiarity

### Installation Steps

```bash
# Clone the repository
git clone https://github.com/username/code-intel-skill.git

# Create symlink to Claude Code skills directory
ln -s /path/to/code-intel-skill ~/.claude/skills/code-intel

# Verify installation
claude skills list | grep code-intel
```

### Uninstallation

```bash
# Remove symlink
rm ~/.claude/skills/code-intel

# Optionally remove cloned repo
rm -rf /path/to/code-intel-skill
```

### Updates

```bash
# Pull latest changes (automatically available via symlink)
cd /path/to/code-intel-skill
git pull
```

**Why this works:** Symlinking the repo directory means users get updates via `git pull` without re-linking. Clean separation between repo location and Claude skills directory.

## File Contents

### README.md

Structure:
```markdown
# code-intel

Strategic code intelligence analysis for Claude Code

## What It Does
[2-3 sentence description]

## Installation
[Installation steps]

## Usage
[Basic invocation and workflow]

## Example Output
[Link to sample report]

## Requirements
[Prerequisites]

## Documentation
[Links to docs/ and examples/]

## License
MIT
```

**Tone:** Direct, clear, assumes technical audience familiar with CLI tools. No marketing fluff. Show value through examples, not claims.

### LICENSE

MIT License - permissive, standard for open source skills/plugins.

### SKILL.md

Moved from `skills/code-intel/SKILL.md` to root. Content unchanged - this is the actual skill artifact that Claude Code loads.

### examples/

Sample intelligence reports demonstrating different use cases:
- `example-fastapi-service.md` - API service analysis
- `example-cli-tool.md` - CLI tool analysis  
- `example-library.md` - Library analysis

Use real or sanitized examples. Purpose: demonstrate report structure, depth, and tone quality.

### docs/design.md

Consolidation of `docs/superpowers/specs/2026-04-26-code-intel-design.md` with internal workflow context removed.

Content:
- What the skill does
- How it works (7-step process)
- Design decisions
- Report structure and template
- Persona and tone guidelines
- Fidelity rules

**Target audience:** Contributors and users who want to understand internals or extend the skill.

## Migration Plan

### Files to Move

1. `skills/code-intel/SKILL.md` → `SKILL.md`
2. `docs/superpowers/specs/2026-04-26-code-intel-design.md` → `docs/design.md` (consolidated)

### Files to Create

1. `README.md` - new, user-facing documentation
2. `LICENSE` - new, MIT license
3. `examples/` - new directory with sample reports

### Files to Remove

1. `docs/superpowers/` - entire directory (keep in git history)
2. `.claude/` - internal tooling directory
3. `docs/code-intel-flask-2026-04-27.md` - move to examples/ or archive

### Git Strategy

**Option A: Clean slate**
- Create new repo with clean structure
- Import only published files
- Fresh commit history

**Option B: Restructure in place**
- Commit restructuring changes
- Keep full git history
- More transparent development history

**Recommendation: Option B** - restructure in place. History shows iterative development, which is authentic. Use clear commit messages for the restructure.

## Documentation Quality Standards

### README.md
- 8th grade reading level
- Direct, jargon-free language
- Installation testable in <2 minutes
- Clear "what it does" in first paragraph
- Example output prominently linked

### docs/design.md
- 8th grade reading level
- Remove "superpowers" internal context
- Focus on architecture and decisions
- Keep 7-step process documentation
- Maintain persona/tone/fidelity sections (these define output quality)

### examples/
- 8th grade reading level
- Real reports or sanitized versions of real runs
- Show variety: different tech stacks, objectives, focus areas
- Demonstrate report quality and depth
- Include metadata footer (provider, model, timestamp)

## Out of Scope

Not included in initial publish:
- .github/ directory (no CI, issue templates)
- CONTRIBUTING.md (add if contributions arrive)
- Versioning/changelog (add when needed)
- Package manager integration (npm/pip) - git clone only for now
- Plugin metadata file (plugin.json) - not needed for symlink install

## Success Criteria

A user should be able to:
1. Find the repo via GitHub search for "Claude Code codebase analysis"
2. Understand what it does from README in <30 seconds
3. Install and run their first analysis in <5 minutes
4. Evaluate output quality from examples before installing
5. Pull updates via `git pull` without re-configuring

Documentation should:
1. Match or exceed counsel-this quality standards
2. Provide clear examples of output
3. Explain design decisions for contributors
4. Use direct, jargon-free language throughout
