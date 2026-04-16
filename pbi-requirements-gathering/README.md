# Power BI Requirements Gathering — Claude Skill

> Most Power BI projects don't fail during the build.  
> They fail in the first two weeks because nobody asked the right questions.

This Claude skill fixes that.

Built by a senior Power BI consultant. Based on real project failures.

---

## What it does

A structured, conversation-driven requirements gathering assistant for Power BI projects. Claude asks the right questions — one at a time — across 10 phases, flags risks as they surface, and produces a portable requirements document your whole team can use.

For a small to mid-size project, the full conversation takes between **half an hour and two hours**, depending on how quickly you can confirm answers. Large enterprise projects take longer.

---

## 10 Phases

| # | Phase | What it catches |
|---|-------|----------------|
| 1 | Business Context & Sponsorship | Vague scope, weak sponsor, missing decision makers |
| 2 | Data & Sources | Dirty data, missing ownership, access problems |
| 3 | Data Modelling & Semantic Layer | Flat tables, no star schema, missing naming conventions |
| 4 | Performance & Scale | Wrong storage mode, volume surprises, high cardinality |
| 5 | Admin & Infrastructure | Refresh limits, gateway issues, Fabric readiness |
| 6 | Report & Visual Requirements | Scope creep, KPI conflicts, mobile gaps |
| 7 | Security, Access & Licensing | RLS design, viewer count, licensing surprises |
| 8 | Integration & Business Logic | KPI definition conflicts, currency, fiscal year |
| 9 | Governance & Workspace Standards | Naming conventions, version control, dataset certification |
| 10 | Change Management & Adoption | No adoption plan, missing training, resistance not addressed |

---

## Outputs

**1. Portable Markdown file** — saved locally, paste back into any Claude session to resume, update, or hand off. Includes a Quick Summary at the top: answered questions, red flags, overall readiness score, and recommended next step.

**2. HTML Requirements Summary** — clean, client-ready output with a readiness score, phase progress bars, collapsible phase sections, and a colour-coded risk register.

---

## How to use

### Option A — Cowork or Claude Code (recommended)

1. Install this skill (download `pbi-requirements-gathering.skill`, add via Settings → Skills)
2. Start a new conversation and say:  
   *"Start Power BI requirements gathering for [project name]"*
3. Claude introduces the process and begins Phase 1
4. Answer one question at a time — Claude flags risks inline
5. Save the markdown file after each phase
6. Paste it back next session to resume

### Option B — Manual

1. Copy `SKILL.md` contents into a Claude system prompt
2. Start a conversation as above

---

## Resuming a session

Paste your saved `requirements.md` at the start of any new Claude session:

*"Here are my requirements so far: [paste markdown]. Pick up where we left off."*

Claude reads your progress, preferences, and open flags — and continues from the right phase.

---

## File structure

```
pbi-requirements-gathering/
├── SKILL.md                          # Claude skill instructions
├── pbi-requirements-gathering.skill  # One-click install bundle
└── references/
    ├── questions.md                  # Full question bank with red flags + tips
    └── requirements-template.md     # Portable markdown template with Quick Summary
```

---

## Related skills

- [pbi-report-builder](https://github.com/lukasreese/powerbi-claude-skills) — Generate Power BI PBIR report files programmatically

---

## About

Built by **Lukas Reese** — Power BI consultant specialising in end-to-end development, data modelling, and IBCS dashboard design.

- 🔗 [linkedin.com/in/lukasreese](https://linkedin.com/in/lukasreese)
- 🌐 [lukasreese.com](https://lukasreese.com)

---

*If this saved you time on a project, a ⭐ on the repo goes a long way.*
