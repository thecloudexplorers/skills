# AI Skills Repository

Reusable AI agent skills for Claude, GitHub Copilot, OpenCode, and other coding agents.

## Repository Structure

```text
skills/
├── README.md
├── LICENSE
├── AGENTS.md
├── skills/
│   ├── caveman/                       # Ultra-compressed communication mode
│   ├── diagnose/                      # Disciplined diagnosis loop for hard bugs
│   ├── grill-me/                      # Relentless interview to stress-test a plan
│   ├── grill-with-docs/               # Grilling session against existing domain docs
│   ├── handoff/                       # Compact conversation into a handoff document
│   ├── improve-codebase-architecture/ # Find architecture deepening opportunities
│   ├── plan-to-markdown/              # Convert plans to structured Markdown docs
│   ├── powershell-style/              # PowerShell scripting conventions
│   ├── prototype/                     # Build throwaway prototypes to flesh out designs
│   ├── setup-matt-pocock-skills/      # Bootstrap agent skill context in a repo
│   ├── tdd/                           # Test-driven development red-green-refactor loop
│   ├── to-issues/                     # Break plans into independently-grabbable issues
│   ├── to-prd/                        # Turn conversation context into a PRD
│   ├── triage/                        # Triage issues through a state machine
│   ├── write-a-skill/                 # Create new agent skills
│   └── zoom-out/                      # Get broader context on unfamiliar code
├── templates/
│   └── skill-template/       # Scaffold for new skills
└── examples/
    └── install-pi.md         # End-to-end setup walkthrough
```

Each skill lives in its own folder:

```text
skills/<skill-name>/
├── SKILL.md          # Main instructions (required)
├── references/       # Supplementary docs and patterns
└── scripts/          # Utility scripts (optional)
```

## Simple Usage

Copy the `skills/` folder into your repository under `.ai/skills/`.

## Synchronized Usage with Git Subtree

Keep skills up to date across projects using `git subtree`.

```bash
# Add remote once
git remote add ai-skills https://github.com/thecloudexplorers/ai-skills.git

# Import skills folder
git subtree add --prefix=.ai/skills ai-skills main --squash

# Pull future updates
git fetch ai-skills
git subtree pull --prefix=.ai/skills ai-skills main --squash

# Push local changes back
git subtree push --prefix=.ai/skills ai-skills main
```

See [examples/install-pi.md](examples/install-pi.md) for a walkthrough.

## Adding a New Skill

1. Copy [templates/skill-template/SKILL.md](templates/skill-template/SKILL.md) to `skills/<your-skill>/SKILL.md`.
2. Fill in the frontmatter `name` and `description` fields.
3. Add reference files under `skills/<your-skill>/references/` if needed.
4. Update `AGENTS.md` if the skill should be auto-loaded.
