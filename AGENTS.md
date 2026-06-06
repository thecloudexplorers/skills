# Agent Instructions

Reusable skills are stored in:

```
skills/
```

Before executing a task, check whether a relevant skill exists in `skills/`.
Each skill is a folder containing a `SKILL.md` file with task-specific instructions, examples, and constraints.

Additional resources per skill:

- `references/` — supplementary patterns and examples
- `scripts/` — utility scripts for deterministic operations

## Available Skills

| Skill | Description |
|-------|-------------|
| [caveman](skills/caveman/SKILL.md) | Ultra-compressed communication mode (~75% fewer tokens) |
| [diagnose](skills/diagnose/SKILL.md) | Disciplined diagnosis loop for hard bugs and performance regressions |
| [grill-me](skills/grill-me/SKILL.md) | Relentless interview to stress-test a plan or design |
| [grill-with-docs](skills/grill-with-docs/SKILL.md) | Grilling session that challenges plans against existing domain docs |
| [handoff](skills/handoff/SKILL.md) | Compact the current conversation into a handoff document |
| [improve-codebase-architecture](skills/improve-codebase-architecture/SKILL.md) | Find architecture deepening opportunities in a codebase |
| [plan-to-markdown](skills/plan-to-markdown/SKILL.md) | Convert plans into structured Markdown documents |
| [powershell-style](skills/powershell-style/SKILL.md) | PowerShell scripting conventions and structure |
| [prototype](skills/prototype/SKILL.md) | Build throwaway prototypes to flesh out a design |
| [setup-matt-pocock-skills](skills/setup-matt-pocock-skills/SKILL.md) | Bootstrap agent skill context (issue tracker, labels, domain docs) in a repo |
| [tdd](skills/tdd/SKILL.md) | Test-driven development with red-green-refactor loop |
| [to-issues](skills/to-issues/SKILL.md) | Break a plan or PRD into independently-grabbable issues |
| [to-prd](skills/to-prd/SKILL.md) | Turn conversation context into a PRD on the issue tracker |
| [triage](skills/triage/SKILL.md) | Triage issues through a state machine driven by triage roles |
| [write-a-skill](skills/write-a-skill/SKILL.md) | Create new agent skills with proper structure |
| [zoom-out](skills/zoom-out/SKILL.md) | Get broader context or a higher-level perspective on unfamiliar code |
| [openspec-apply-change](skills/openspec-apply-change/SKILL.md) | Implement tasks from an OpenSpec change |
| [openspec-archive-change](skills/openspec-archive-change/SKILL.md) | Archive a completed change in the OpenSpec experimental workflow |
| [openspec-explore](skills/openspec-explore/SKILL.md) | Thinking partner for exploring ideas and clarifying requirements before a change |
| [openspec-propose](skills/openspec-propose/SKILL.md) | Propose a new change with design, specs, and tasks in one step |
| [openspec-propose-arch](skills/openspec-propose-arch/SKILL.md) | Propose a change from an architecture/business/product owner viewpoint |
| [openspec-reverse-engineer](skills/openspec-reverse-engineer/SKILL.md) | Reverse engineer an existing codebase into OpenSpec format |

# Agent Behaviour

Behavioral guidelines to reduce common LLM coding
mistakes. Merge with project-specific instructions
as needed.

Tradeoff: These guidelines bias toward caution
over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

Don't assume. Don't hide confusion. Surface tradeoffs.

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them.
- If a simpler approach exists, say so.
- If something is unclear, stop. Name what's confusing.

## 2. Simplicity First

Minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No “flexibility” that wasn't requested.
- No error handling for impossible scenarios.
- If 200 lines could be 50, rewrite it.

## 3. Surgical Changes

Touch only what you must. Clean up only your own mess.

- Don't “improve” adjacent code or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice dead code, mention it — don't delete it.

## 4. Goal-Driven Execution

Define success criteria. Loop until verified.

Transform tasks into verifiable goals:

- “Add validation” → “Write tests, then make them pass”
- “Fix the bug” → “Reproduce it in a test, then fix”
- “Refactor X” → “Ensure tests pass before and after”
