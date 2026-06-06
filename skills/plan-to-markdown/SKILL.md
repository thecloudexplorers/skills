---
name: plan-to-markdown
description: Convert a verbal or structured plan into a clean, shareable Markdown document with sections, checklists, and decision rationale. Use when the user wants to document a plan, write a proposal, capture a design decision, or produce a handoff document.
---

# Plan to Markdown

## Quick start

Given a plan (verbal, bullet-point, or structured), produce a Markdown document with:

1. A clear title and one-sentence summary
2. Sections matching the plan's logical groupings
3. Checklists for action items
4. Decision rationale where choices were made

See [references/plan-template.md](references/plan-template.md) for the full template.

## Workflow

1. **Identify the audience** — technical team, stakeholders, or mixed? This drives jargon level.
2. **Extract structure** — find the phases, tasks, decisions, and dependencies in the input.
3. **Map to sections**:
   - Background / context
   - Goals and non-goals
   - Approach / phases
   - Open questions
   - Action items (checklist)
4. **Write** using the template as a scaffold.
5. **Review** — every checklist item should have an owner and due date if known.

## Section guidelines

### Background

One short paragraph. Why does this plan exist? What problem does it solve?

### Goals and non-goals

Two short lists. Non-goals prevent scope creep — include at least one.

### Approach

Use numbered phases for sequential work, or bullet sections for parallel tracks. Each phase gets:
- What happens
- Who is responsible
- Exit criteria or definition of done

### Open questions

A numbered list of unresolved decisions. Each question should state who will resolve it and by when.

### Action items

A markdown checklist:

```markdown
- [ ] @alice Set up the dev environment by 2026-06-10
- [ ] @bob Draft the API contract by 2026-06-12
- [ ] @carol Review security requirements — no deadline yet
```

## Constraints

- Keep the document under 2 pages (≈ 600 words) unless the plan is genuinely complex.
- Use plain English headings; avoid jargon in section titles.
- Do not include information that was not in the original plan without flagging it as an assumption.
