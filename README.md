# Claude Code Skills

Open-source skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — Anthropic's CLI for Claude.

Skills are reusable instruction sets that extend Claude Code's capabilities. Drop them into `~/.claude/skills/` and they're available in every session.

## Available Skills

| Skill | Description | Use When |
|-------|-------------|----------|
| [reflect-and-improve](skills/reflect-and-improve/SKILL.md) | Self-improvement loop that reviews errors, tracks permissions, captures preferences, and writes learnings to persistent memory | After significant work, error recovery, or session breakpoints |

## Installation

### Quick Install (single skill)

```bash
# Create the skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Copy the skill you want
cp -r skills/reflect-and-improve ~/.claude/skills/
```

### Install All Skills

```bash
# Clone the repo
git clone https://github.com/aroyburman-codes/claude-skills.git

# Copy all skills
cp -r claude-skills/skills/* ~/.claude/skills/
```

### Verify Installation

Skills are automatically detected by Claude Code. After copying, start a new session and the skill will appear in your available skills list.

## How Skills Work

A Claude Code skill is a markdown file (`SKILL.md`) placed in `~/.claude/skills/<skill-name>/`. It contains:

1. **YAML frontmatter** — `name` and `description` fields that tell Claude Code when to activate the skill
2. **Instructions** — Structured guidance that Claude follows when the skill triggers
3. **Examples** — Concrete patterns so Claude knows exactly what output to produce

Skills are loaded into Claude's context when their trigger conditions match, giving Claude specialized knowledge and workflows it wouldn't have by default.

## Creating Your Own Skills

```bash
mkdir -p ~/.claude/skills/my-skill
```

Then create `SKILL.md`:

```markdown
---
name: my-skill
description: Use when [trigger condition] to [what it does]
---

# My Skill

## Overview
What this skill does and why.

## When to Use
Trigger conditions.

## Instructions
Step-by-step guidance for Claude.
```

The `description` field is critical — it tells Claude Code when to activate the skill. Write it as a clear trigger condition.

## Contributing

PRs welcome. To add a skill:

1. Create `skills/<your-skill-name>/SKILL.md`
2. Follow the YAML frontmatter format (see existing skills)
3. Include clear trigger conditions in the description
4. Add your skill to the table in this README
5. Open a PR

## License

MIT
