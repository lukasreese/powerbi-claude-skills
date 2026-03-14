# Power BI Skills for Claude Code

A collection of [Claude Code](https://claude.ai/code) skills that let Claude build Power BI reports, audit data models, and generate visuals programmatically — no manual JSON or DAX required.

## Available Skills

| Skill | Description | Status |
|-------|-------------|--------|
| [**pbir-report-builder**](./pbir-report-builder/) | Generate report pages, visuals, and IBCS variance charts by writing PBIR JSON directly into PBIP projects | Available |
| [**pbip-dependency-analyzer**](./pbip-dependency-analyzer/) | Find unused measures, run impact analysis, and audit semantic model quality | Coming soon |

## Quick Install

```bash
# Clone this repo
git clone https://github.com/lukasreese/powerbi-claude-skills.git

# Copy the skill you want to your Claude Code skills directory
cp -r powerbi-claude-skills/pbir-report-builder ~/.claude/skills/
```

Or for project-specific use:

```bash
cp -r powerbi-claude-skills/pbir-report-builder ./.claude/skills/
```

> **Important:** Copy the entire skill folder — the `SKILL.md` and `references/` directory work together.

See each skill's own README for detailed setup and usage instructions.

## What Are Claude Code Skills?

Skills are folders containing a `SKILL.md` file (with optional reference materials) that teach Claude Code how to perform specialized tasks. Once installed to `~/.claude/skills/`, Claude automatically detects and uses them when relevant.

## About

Built by [Lukas Reese](https://github.com/lukasreese) — Power BI consultant and Claude Code skill developer.
