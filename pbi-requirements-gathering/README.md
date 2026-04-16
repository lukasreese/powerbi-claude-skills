# 🟠 Power BI Requirements Gathering — Claude Skill

> Most Power BI projects don't fail during the build.  
> They fail in the first two weeks because nobody asked the right questions.

This Claude skill fixes that.

---

## What it does

A structured, conversation-driven requirements gathering assistant for Power BI projects. Claude asks the right questions — one at a time — across 8 phases, flags risks as they surface, and produces a portable requirements document your whole team can use.

Built by a senior Power BI consultant. Based on real project failures.

---

## 8 Phases

| # | Phase | What it catches |
|---|-------|----------------|
| 1 | Business Context | Vague scope, missing decision makers |
| 2 | Data & Sources | Dirty data, missing ownership, access problems |
| 3 | Performance & Scale | Wrong storage mode, volume surprises |
| 4 | Admin & Infrastructure | Refresh limits, gateway issues, Fabric readiness |
| 5 | Report & Visual Requirements | Scope creep, KPI conflicts, mobile gaps |
| 6 | Security, Access & Licensing | RLS design, viewer count, licensing surprises |
| 7 | Integration & Business Logic | KPI definition conflicts, currency, fiscal year |
| 8 | Project & Delivery | No sign-off owner, missing UAT, go-live risk |

---

## Outputs

**1. Portable Markdown file** — saved locally, paste back into any Claude session to resume, update, or hand off. Human-readable and directly editable.

**2. HTML Requirements Summary** — clean, client-ready output with completeness scores per phase and a colour-coded risk register.

---

## How to use

### Option A — Claude.ai (recommended)

1. Install this skill in your Claude environment
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

Claude reads your progress, preferences, and flags — and continues from the right phase.

---

## File structure

```
pbi-requirements-gathering/
├── SKILL.md                          # Claude skill instructions
└── references/
    ├── questions.md                  # Full question bank with red flags + tips
    └── requirements-template.md     # Blank template to start a new project
```

---

## Related skills

- [pbi-report-builder](https://github.com/lukasreese) — Generate Power BI PBIR report files programmatically
- More Power BI × Claude skills coming — follow for updates

---

## About

Built by **Lukas Reese** — Power BI consultant and developer specialising in Claude-powered BI workflows.

- 🔗 [LinkedIn](https://linkedin.com/in/lukasreese)
- 🌐 [lukasreese.com](https://lukasreese.com)

---

*If this saved you time on a project, a ⭐ on the repo goes a long way.*
