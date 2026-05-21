---
name: azdo-sync-specs
description: "Sync OpenSpec specs to Azure DevOps work items using MCP ADO tools. Use when you want to manually create or update ADO work items from specification files."
argument-hint: 'Spec name or "all" to sync multiple specs'
---

# Azure DevOps - Sync Specs

Sync OpenSpec specification files to Azure DevOps work items using the `ado-work-items` MCP server.

Content transformation is handled by the **`azdo-openspec-to-workitems`** skill, which maps spec content to the correct ADO work item hierarchy (Epic → Feature → User Story → Task → Test Case). This skill then executes that hierarchy against ADO via MCP tools.

## When to Use

- Create Azure DevOps work items from OpenSpec specs
- Update existing work items when specs change
- One-time sync or testing ADO integration

## Prerequisites

- The `ado-work-items` MCP server is configured and enabled in Claude Code

Authentication is handled automatically by the MCP server — no PAT token is required.

## Procedure

### 1. Resolve Project and Backlog Configuration

Read `openspec/config.yaml` and check for `ado.project`, `ado.team`, and `ado.areaPath`.

**If `ado.project` is missing:**

```
mcp__ado-work-items__core_list_projects
```

Show the project list and ask the user to select one.

**If `ado.team` or `ado.areaPath` is missing:**

```
mcp__ado-work-items__core_list_project_teams
- project: <project-name>
```

Show the team list and ask the user: _"Which team's backlog should these work items be synced to?"_

Derive `areaPath` as `<project>\<team-name>`. Confirm with the user if the team may have a non-standard area path.

**Persist to `openspec/config.yaml`** any newly selected values (`project`, `team`, `areaPath`) so future runs skip this prompt. Use the Read and Edit tools to update the file — preserve all existing fields.

Use the resolved project, team, and areaPath for all subsequent steps.

### 2. Discover Specs to Sync

**If user specified a change name:**

- Look under `openspec/changes/<change-name>/specs/` for `spec.md` files
- Also check for `proposal.md` and `tasks.md` at the change root

**If user said "all" or no spec specified:**

- Find all spec.md files:
  ```bash
  find openspec/changes/*/specs/ -name "spec.md" -type f
  find openspec/specs -name "spec.md" -type f
  ```
- Show list and confirm with user before proceeding

### 3. Derive and Confirm Labels for All Changes

For each distinct change being synced, produce a proposed label then **ask the user to confirm before proceeding**.

#### 3a. Derive the label (per change)

Produce a label that will prefix every work item title and be added as a tag. The label must be as short as possible while still being unique and human-readable.

**Algorithm:**

1. Start with the change name (folder name under `openspec/changes/`, e.g. `multi-region-network-deployment`).
2. Strip any trailing segment that is purely numeric or a version tag (e.g. `-v2`, `-2026`).
3. If the name has three or more hyphen-separated words, keep only the first two (e.g. `multi-region-network-deployment` → `multi-region`). If it already has one or two words, keep it as-is.
4. If a `ado.label` value already exists in any spec frontmatter for this change, use that as the starting point instead of re-deriving.
5. The kebab-case result is the **tag label** (e.g. `multi-region`).
6. Convert the tag label to Title Case for use in titles: capitalise the first letter of each hyphen-separated word and replace hyphens with spaces (e.g. `multi-region` → `Multi Region`).

This gives two forms, both derived from the same base:

- **`<label-title>`** — used in work item titles (e.g. `Multi Region`)
- **`<label-kebab>`** — used in tags (e.g. `multi-region`)

#### 3b. Confirm labels with the user

Before reading spec content or writing anything to ADO, present all proposed labels in a table and ask for confirmation:

```
The following labels will be used to prefix all work item titles and tags.
Please confirm or provide a replacement for any label you want to change.

| Change                          | Tag label (kebab)     | Title prefix            |
|---------------------------------|-----------------------|-------------------------|
| multi-region-network-deployment | multi-region          | [Multi Region]          |
| cloud-governance-foundation     | cloud-governance      | [Cloud Governance]      |
| workload-onboarding             | workload-onboarding   | [Workload Onboarding]   |

Reply "ok" to confirm all, or specify changes:
  e.g. "change multi-region to hub-spoke"
       "use [Platform Network] for multi-region-network-deployment"
```

Wait for the user's reply before continuing. Apply any corrections they request. **Do not proceed to step 4 until the user has confirmed the labels.**

If only one change is being synced, still show the single-row table and ask for confirmation — the user may want a shorter or different label.

### 4. Read Spec Content and Check Existing State

For each spec file:

- Read the entire file content
- Parse frontmatter: check for existing `ado.hierarchy` entries (IDs from a prior sync)
- Note which work items already exist vs need to be created

### 5. Transform to ADO Work Item Structure

**Apply the `azdo-openspec-to-workitems` skill** to the spec content.

Pass the full spec markdown as input. The skill produces a structured work item plan:

```
## Proposed Work Item Hierarchy
- Epic: <name>          (only if scope warrants it)
  - Feature: <name>
    - User Story: <story-title>
        - Task: <task-title>
        - Test Case: <test-title>

## Work Items
## Feature: <name>
### Description
...
### Business / Technical Value
...

## User Story: <name>
### User Story
As a <role>, I want <capability>, so that <outcome>.
### Acceptance Criteria
- [ ] Given ..., when ..., then ...

## Task: <name>
### Description / Checklist
...

## Test Case: <name>
### Test Steps / Expected Result
...

## Traceability Matrix
| OpenSpec Item | Azure DevOps Item | Type | Notes |
```

**If the user has already run `azdo-openspec-to-workitems`** and provides the output, use that directly — skip re-running the transformation.

**Before proceeding to step 6**, show the proposed hierarchy to the user and confirm. This is the last checkpoint before writing to ADO.

### 6. Create Work Items — Parent First

Create work items top-down so parents exist before children are linked. For each item in the hierarchy:

**Check for existing ID first:**

- If `ado.hierarchy` frontmatter has an ID for this item → go to step 7 (update)
- If no ID → create new

**Title format:** Every work item title must be prefixed with the PascalCase label in square brackets:

```
[<label-title>] <title from skill output>
```

Example: `[Multi Region] Enforce Hub-and-Spoke Networking`

**Create new work item:**

```
mcp__ado-work-items__wit_create_work_item
- project: <project-name>
- workItemType: Epic | Feature | User Story | Task | Test Case
- fields:
  - name: System.Title
    value: "[<label-title>] <title from skill output>"
  - name: System.Description
    value: <description from skill output>
    format: Html
  - name: Microsoft.VSTS.Common.AcceptanceCriteria   (User Story only)
    value: <acceptance criteria from skill output>
    format: Html
  - name: System.AreaPath
    value: <ado.areaPath from config>          # routes item to the correct team backlog
  - name: System.Tags
    value: "openspec; <change-name>; <label-kebab>"
```

Convert the skill's markdown content to HTML before setting fields:

- `## Heading` → `<h2>Heading</h2>`
- Paragraphs → `<p>...</p>`
- Bullet lists → `<ul><li>...</li></ul>`
- Checkboxes `- [ ] item` → `<ul><li>[ ] item</li></ul>`
- `**bold**` → `<strong>bold</strong>`
- Escape `<`, `>`, `&` as HTML entities

**Creation order:**

1. Epic (if present)
2. Feature(s)
3. User Stories (under each Feature)
4. Tasks and Test Cases (under each User Story)

### 7. Update Existing Work Items

For items that already have an ADO ID in frontmatter:

```
mcp__ado-work-items__wit_update_work_item
- id: <existing-id>
- updates:
  - path: /fields/System.Description
    value: <updated HTML content>
  - path: /fields/Microsoft.VSTS.Common.AcceptanceCriteria
    value: <updated HTML criteria>
  - path: /fields/System.Tags
    value: "openspec; <change-name>; <label-kebab>"
```

Preserve `System.State` unless the spec title has explicitly changed. If the title changed, update it (keeping the `[<label-title>]` prefix).

### 8. Link Parent-Child Hierarchy

After all work items are created, establish the hierarchy using:

```
mcp__ado-work-items__wit_work_items_link
- sourceId: <parent-id>
- targetId: <child-id>
- linkType: System.LinkTypes.Hierarchy-Forward
```

Link order:

- Epic → Feature(s)
- Feature → User Stories
- User Story → Tasks
- User Story → Test Cases

Alternatively use `mcp__ado-work-items__wit_add_child_work_items` when creating children under a known parent in bulk.

### 9. Write IDs Back to Spec Frontmatter

After successful creation, update the spec file frontmatter to record all created IDs:

```yaml
---
ado:
  hierarchy:
    epic:
      id: 10
      url: https://dev.azure.com/<org>/<project>/_workitems/edit/10
    feature:
      id: 11
      url: https://dev.azure.com/<org>/<project>/_workitems/edit/11
    userStories:
      - id: 12
        title: "Story title"
        url: https://dev.azure.com/<org>/<project>/_workitems/edit/12
    tasks:
      - id: 13
        title: "Task title"
        url: https://dev.azure.com/<org>/<project>/_workitems/edit/13
    testCases:
      - id: 14
        title: "Test case title"
        url: https://dev.azure.com/<org>/<project>/_workitems/edit/14
  syncedAt: "2026-05-21T10:00:00Z"
  label: "<label>"
---
```

Place at the very beginning of the spec file, before any markdown content. Preserve any other existing frontmatter fields.

### 10. Report Results

Provide a summary:

```
Synced: openspec/changes/multi-region-network-deployment/specs/hub-spoke/spec.md
Label: [Multi Region]  (tag: multi-region)

Work Items Created:
  Feature #11  [Multi Region] Enforce Hub-and-Spoke Networking
  User Story #12  [Multi Region] Configure VNet peering
  User Story #13  [Multi Region] Validate routing tables
  Task #14  [Multi Region] Create Bicep hub network module
  Test Case #15  [Multi Region] Hub connectivity validation

View in ADO: https://dev.azure.com/<org>/<project>/_backlogs/backlog
```

## Updating Already-Synced Specs

When a spec has existing IDs in frontmatter:

1. Read current spec content
2. Re-derive the label (or read `ado.label` from frontmatter if present)
3. Run `azdo-openspec-to-workitems` on the updated content
4. Compare new structure with existing IDs:
   - Items with matching titles → update description/criteria and ensure tags include `<label-kebab>`
   - New items (no existing ID) → create (with `[<label-title>]` prefix) and link
   - Items removed from spec → add comment to work item noting removal; do not delete
5. Update frontmatter with any new IDs

## Error Handling

**Transformation produces unexpected structure:**

- Review the `azdo-openspec-to-workitems` output before proceeding
- Adjust the input or re-run with more context if hierarchy looks wrong

**Work item creation failed:**

- Check work item type exists in the project (some projects use custom types)
- Try with `System.Title` only first, then add fields
- Verify field names match the project's ADO process template

**Parent not found when linking:**

- Ensure parent was created successfully before linking children
- Fall back to creating as a standalone item, then manually link

**Authentication failed:**

- Confirm the `ado-work-items` MCP server is enabled
- Test with: `mcp__ado-work-items__core_list_projects`

## MCP Tools Used

- `mcp__ado-work-items__core_list_projects` — Resolve target project
- `mcp__ado-work-items__core_list_project_teams` — List teams to select target backlog
- `mcp__ado-work-items__wit_create_work_item` — Create new work item
- `mcp__ado-work-items__wit_update_work_item` — Update existing work item
- `mcp__ado-work-items__wit_get_work_item` — Verify work item exists
- `mcp__ado-work-items__wit_work_items_link` — Link parent-child
- `mcp__ado-work-items__wit_add_child_work_items` — Create child items in bulk
- `mcp__ado-work-items__wit_query_by_wiql` — Check for existing items

## Related Skills

- `azdo-openspec-to-workitems` — Transforms spec content into ADO work item structure (used in step 5)
- `azdo-query` — Query and manage synced work items
- `azdo-setup` — Configure ADO integration
