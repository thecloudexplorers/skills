---
name: azdo-sync-wiki
description: 'Sync OpenSpec documentation (proposal, design) to Azure DevOps Wiki using MCP tools. Use when publishing OpenSpec changes to ADO Wiki for team visibility and documentation.'
argument-hint: 'Change name or path to sync documentation'
---

# Azure DevOps - Sync Documentation to Wiki

Publish OpenSpec documentation files (proposal.md, design.md, specs) to Azure DevOps Wiki for team visibility and collaboration.

## When to Use

- Publish OpenSpec proposals to ADO Wiki
- Sync design documents to ADO Wiki
- Create documentation structure in ADO Wiki that mirrors OpenSpec changes
- Update wiki pages when OpenSpec documentation changes
- Generate a specification for this skill (always create specs when building new skills)

## Prerequisites

- The `ado-work-items` MCP server is configured and enabled in Claude Code
- The target ADO project has a wiki (either project wiki or code wiki)

Authentication is handled automatically by the MCP server — no PAT token is required.

## Procedure

### 1. Resolve Project Configuration

Check `openspec/config.yaml` for `ado.project`. If not present, call:

```
mcp__ado-work-items__core_list_projects
```

Show the project list and ask the user to select one. Use the selected project for all subsequent steps.

### 2. Check Wiki Availability

List wikis for the project:

```
mcp__ado-work-items__wiki_list_wikis
- project: <project-name>
```

**If no wikis found:**
- Inform the user that a wiki must be created in ADO first
- Guide: Project Settings → Overview → Enable Wiki, or create a project wiki from the Wiki section
- Stop and wait for wiki creation

**If multiple wikis found:**
- Show list with names/identifiers
- Ask user which wiki to use (default to the first project wiki)
- Store the selected wiki identifier for subsequent operations

### 3. Discover Documentation to Sync

Determine what needs to be synced based on the user request:

**If user specified a change name:**
- Look for documentation in `openspec/changes/<change-name>/`
- Find: `proposal.md`, `design.md`, and spec files in `specs/` subdirectories

**If user specified a path:**
- Read files from that path
- Support: `proposal.md`, `design.md`, or `spec.md` files

**If no specific request:**
- List available changes: `ls openspec/changes/`
- Show options and ask user which change to sync

### 4. Determine Wiki Structure

Create a hierarchical wiki structure that mirrors OpenSpec organization:

**Wiki Path Pattern:**
```
/OpenSpec
  /<change-name>
    /Proposal
    /Design
    /Specifications
      /<spec-name>
```

**Example:**
```
/OpenSpec
  /cloud-governance-foundation
    /Proposal
    /Design
    /Specifications
      /governance-policy
```

### 5. Read and Prepare Content

For each file to sync:

**Read the file** using the Read tool to get complete content.

**Prepare markdown content:**
- Wiki pages use standard markdown — no conversion needed
- Keep headings, lists, and code blocks as-is
- Add a metadata footer:
  ```markdown
  ---
  *Synced from OpenSpec on <date>*
  *Source: openspec/changes/<path>/<file>*
  ```

### 6. Create or Update Wiki Pages

Use `mcp__ado-work-items__wiki_create_or_update_page` for each document:

**Parameters:**
- `wikiIdentifier`: The wiki ID from step 2
- `project`: Project name
- `path`: The wiki path (e.g., `/OpenSpec/cloud-governance-foundation/Proposal`)
- `content`: The markdown content from step 5

**Process each file:**

1. **Proposal** → `/OpenSpec/<change-name>/Proposal`
2. **Design** → `/OpenSpec/<change-name>/Design`
3. **Specs** → `/OpenSpec/<change-name>/Specifications/<spec-name>`

**Result handling:**
- Tool returns page details with URL on success
- Collect URLs for summary report
- Handle errors gracefully (permissions, invalid paths, etc.)

### 7. Create Index/Navigation Pages

After syncing content, create navigation pages:

**Main OpenSpec Index** (`/OpenSpec`):
```markdown
# OpenSpec Documentation

This wiki contains specifications and design documentation synced from the OpenSpec repository.

## Changes

- [cloud-governance-foundation](/OpenSpec/cloud-governance-foundation/Proposal)
- [enterprise-network-governance](/OpenSpec/enterprise-network-governance/Proposal)
...
```

**Change Index** (`/OpenSpec/<change-name>`):
```markdown
# <Change Title>

## Documents

- [Proposal](/OpenSpec/<change-name>/Proposal)
- [Design](/OpenSpec/<change-name>/Design)
- [Specifications](/OpenSpec/<change-name>/Specifications)

## Related Work Items

- Feature: [#<id>](https://dev.azure.com/<org>/<project>/_workitems/edit/<id>)
...
```

### 8. Report Results

Generate a summary report:

```markdown
Synced OpenSpec Documentation to ADO Wiki

**Change:** <change-name>

**Pages Created/Updated:**
- Proposal: <wiki-url>
- Design: <wiki-url>
- Spec (<spec-name>): <wiki-url>

**Wiki:** <wiki-name>
**Project:** <project-name>
```

## Creating Specifications for Skills

**When building new skills or features, always create an OpenSpec specification:**

### Specification Creation Workflow

1. **Analyze the skill requirements:**
   - What problem does the skill solve?
   - What are the key capabilities?
   - What scenarios should it support?

2. **Create the spec directory structure:**
   ```
   openspec/changes/<feature-name>/
   ├── proposal.md      # Problem statement and solution
   ├── design.md        # Architecture and approach (if complex)
   ├── tasks.md         # Implementation tasks
   └── specs/
       └── <feature-name>/
           └── spec.md  # Detailed requirements and scenarios
   ```

3. **Write the specification:**
   - Use OpenSpec format with requirements and scenarios
   - Include SHALL statements for mandatory behavior
   - Add acceptance criteria and test scenarios
   - Document dependencies and constraints

4. **Sync to ADO:**
   - Use `azdo-sync-specs` skill to create work items
   - Link work items in spec frontmatter
   - Sync documentation to wiki using this skill

### Spec Template

```markdown
---
ado:
  # Work item metadata will be added after sync
---

# Specification: <Skill Name>

## Overview

Brief description of what the skill accomplishes.

## Requirements

### REQ-1: <Requirement Title>

As a user, I want to <user story> so that <benefit>.

The system SHALL <mandatory behavior>.

#### Scenarios

**Scenario 1.1: <Scenario Name>**
- Given <precondition>
- When <action>
- Then <expected outcome>

### REQ-2: <Next Requirement>
...

## Dependencies

- List any dependencies on other skills, tools, or configurations

## Constraints

- Document any limitations or constraints
```

## Error Handling

**Wiki not found:**
- Guide user to create wiki in ADO
- Navigate to: `https://dev.azure.com/<org>/<project>/_wiki`

**Permission errors:**
- Confirm MCP server is authenticated with an account that has Wiki (Contribute) permission
- Ask the project admin to grant access

**Invalid paths:**
- Validate wiki paths (no special characters except `/`, `-`, `_`)
- Auto-correct common issues (spaces → dashes)

**Content too large:**
- Wiki pages have size limits (~1 MB)
- Split large specs into multiple linked pages

## Tips

- **Organize by change:** Keep related docs together under one change folder
- **Use relative links:** Link between wiki pages using `/OpenSpec/...` paths
- **Update regularly:** Re-sync when proposal/design changes
- **Add metadata:** Include sync date and source path in the footer
- **Create specs first:** Always document skills with OpenSpec before implementation
