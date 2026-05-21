# AI Skills Repository Template

Reusable AI agent skills for Claude, GitHub Copilot, OpenCode, and other coding agents.

This repository stores shared skills in one standard location:

```text
.ai/skills/
```

## Repository Structure

```text
your-repository/
├── .ai/
│   └── skills/
│       └── skill-name/
│           └── SKILL.md
├── AGENTS.md
├── .github/
│   └── copilot-instructions.md
└── README.md
```

## How It Works

`.ai/skills/` is the source of truth for reusable skills.

Agent-specific files should point agents to this directory:

```text
AGENTS.md
.github/copilot-instructions.md
```

Each skill should have its own folder:

```text
.ai/skills/git-subtree/SKILL.md
```

## Simple Usage

Copy the `.ai/skills/` folder into your repository.

Use this when you only need a one-time setup.

## Synchronized Usage with Git Subtree

Use Git Subtree when you want to keep the skills synchronized with this repository.

Add this repository as a remote:

```bash
git remote add ai-skills-template https://github.com/thecloudexplorers/ai-skills.git
```

Add the skills folder to your repository:

```bash
git subtree add --prefix=.ai/skills ai-skills-template main --squash
```

Pull future updates:

```bash
git fetch ai-skills-template
git subtree pull --prefix=.ai/skills ai-skills-template main --squash
```

Push local changes back to this repository:

```bash
git subtree push --prefix=.ai/skills ai-skills-template main
```

You need write access to push changes back.

## Quick Reference

```bash
git remote add ai-skills-template https://github.com/thecloudexplorers/ai-skills.git
git subtree add --prefix=.ai/skills ai-skills-template main --squash

git fetch ai-skills-template
git subtree pull --prefix=.ai/skills ai-skills-template main --squash

git subtree push --prefix=.ai/skills ai-skills-template main
```
