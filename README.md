# Power BI Skills for Claude Code

A collection of [Claude Code](https://claude.ai/code) skills that let Claude build Power BI reports, audit data models, and generate visuals programmatically — no manual JSON or DAX required.

## Available Skills

| Skill | Description | Status |
|-------|-------------|--------|
| [**pbir-report-builder**](./pbir-report-builder/) | Generate report pages, visuals, and IBCS variance charts by writing PBIR JSON directly into PBIP projects | Available |
| [**pbip-dependency-analyzer**](./pbip-dependency-analyzer/) | Find unused measures, run impact analysis, and audit semantic model quality | Coming soon |

## Quick Install

Each skill folder contains a ready-to-use `.skill` file — just download it and add it to Claude:

1. Go to the skill folder (e.g., [`pbir-report-builder/`](./pbir-report-builder/))
2. Click the `.skill` file → **Download raw file** (download icon)
3. Add it to Claude Desktop or Cowork via Settings → Skills

No cloning, no terminal — just download one file and you're ready to go.

> **For developers:** You can also clone the repo and copy the skill folder to `~/.claude/skills/` if you prefer working with the raw files.

See each skill's own README for detailed usage instructions.

## What Are Claude Code Skills?

Skills are folders containing a `SKILL.md` file (with optional reference materials) that teach Claude Code how to perform specialized tasks. Once installed to `~/.claude/skills/`, Claude automatically detects and uses them when relevant.

## About

Built by [Lukas Reese](https://github.com/lukasreese) — Power BI consultant and Claude Code skill developer.
