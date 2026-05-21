---
name: azdo-query
description: 'Query and manage Azure DevOps work items. Use when searching for work items, checking sync status, or managing ADO backlog items.'
argument-hint: 'Search query or work item ID'
---

# Azure DevOps - Query & Management

Search, query, and manage Azure DevOps work items related to OpenSpec specs.

## When to Use

- Find work items synced from specs
- Check if a spec has an associated work item
- Search for work items by keyword
- Verify sync status
- Update work item states
- Link related work items

## Prerequisites

- The `ado-work-items` MCP server is configured and enabled in Claude Code

Authentication is handled automatically by the MCP server — no PAT token is required.

## Resolve Configuration

Before querying, read `openspec/config.yaml` to load `ado.project`, `ado.team`, and `ado.areaPath`. These narrow queries to the correct team backlog.

If `ado.team` is missing, call:
```
mcp__ado-work-items__core_list_project_teams
- project: <project-name>
```
Ask the user which team backlog to scope the query to, then persist the selection to `openspec/config.yaml`.

If the user wants results across all teams, skip the area path filter.

## Common Operations

### 1. Find Work Items for a Spec

**By spec name using WIQL (scoped to configured team backlog):**
```
mcp__ado-work-items__wit_query_by_wiql
- wiql: "SELECT [System.Id],[System.Title] FROM WorkItems WHERE [System.Tags] CONTAINS 'openspec' AND [System.Title] CONTAINS '<spec-name>' AND [System.AreaPath] UNDER '<ado.areaPath>'"
- project: <project-name>
```

Omit the `[System.AreaPath]` clause if no team is configured or the user requested all teams.

**By work item ID from spec frontmatter:**
1. Read spec frontmatter for `ado.workItemId`
2. Use `mcp__ado-work-items__wit_get_work_item` with the ID

### 2. Search Work Items

**By keyword:**
```
mcp__ado-work-items__wit_query_by_wiql
- wiql: "SELECT [System.Id],[System.Title],[System.State] FROM WorkItems WHERE [System.TeamProject] = '<project>' AND [System.Title] CONTAINS 'authentication'"
- project: <project-name>
```

**My work items:**
```
mcp__ado-work-items__wit_my_work_items
- type: "assignedtome"
- includeCompleted: false
```

**Batch fetch by IDs:**
```
mcp__ado-work-items__wit_get_work_items_batch_by_ids
- ids: [1, 2, 3]
- project: <project-name>
```

### 3. Check Work Item Details

Get full work item with all fields:
```
mcp__ado-work-items__wit_get_work_item
- id: <work-item-id>
- expand: "all"
```

Get specific fields only:
```
mcp__ado-work-items__wit_get_work_item
- id: <work-item-id>
- fields: ["System.Title", "System.State", "System.AssignedTo"]
```

### 4. Update Work Item State

Change work item state (e.g., To Do → In Progress → Done):
```
mcp__ado-work-items__wit_update_work_item
- id: <work-item-id>
- updates:
  - path: "/fields/System.State"
    value: "In Progress"
```

Common states:
- Task: `To Do`, `In Progress`, `Done`, `Removed`
- User Story: `New`, `Active`, `Resolved`, `Closed`
- Bug: `New`, `Active`, `Resolved`, `Closed`

### 5. Add Comments

Add a comment to a work item:
```
mcp__ado-work-items__wit_add_work_item_comment
- workItemId: <id>
- text: "Updated from OpenSpec spec sync"
- format: "Markdown"
```

### 6. Link Work Items

Link related work items together:
```
mcp__ado-work-items__wit_work_items_link
- sourceId: <parent-id>
- targetId: <child-id>
- linkType: "System.LinkTypes.Hierarchy-Forward"
```

Common link types:
- `System.LinkTypes.Hierarchy-Forward` — Parent-child
- `System.LinkTypes.Related` — Related items
- `System.LinkTypes.Dependency-Forward` — Depends on

## Find Specs Missing Work Items

To identify specs that haven't been synced yet:

1. List all spec files:
```bash
find openspec/specs -name "spec.md" -type f
find openspec/changes/*/specs -name "spec.md" -type f
```

2. For each spec:
   - Read frontmatter
   - Check for `ado.workItemId`
   - If missing → spec not synced

3. Report which specs need syncing

## Find Orphaned Work Items

Work items whose specs no longer exist:

1. Query work items with "openspec" tag:
```
mcp__ado-work-items__wit_query_by_wiql
- wiql: "SELECT [System.Id],[System.Title],[System.Tags] FROM WorkItems WHERE [System.Tags] CONTAINS 'openspec'"
- project: <project-name>
```

2. For each work item:
   - Extract spec name from tags or description
   - Check if spec file exists in workspace
   - If missing → work item is orphaned

3. Options:
   - Close orphaned work items
   - Update with note about missing spec
   - Archive or remove

## Batch Operations

### Update Multiple Work Items

Use `mcp__ado-work-items__wit_update_work_items_batch` for efficiency:

```
updates: [
  { id: 1, path: "/fields/System.State", value: "Done" },
  { id: 2, path: "/fields/System.State", value: "Done" }
]
```

### Create Child Tasks

Create multiple child work items under a parent:

```
mcp__ado-work-items__wit_add_child_work_items
- parentId: <parent-id>
- workItemType: "Task"
- items:
  - title: "Task 1"
    description: "Details..."
  - title: "Task 2"
    description: "Details..."
```

## Sync Status Report

Generate a comprehensive sync status report:

### 1. Count Synced Specs

- Count specs with `ado.workItemId` in frontmatter
- Total specs vs synced specs ratio

### 2. Work Item States

For all synced work items, count by state:
- To Do: X
- In Progress: Y
- Done: Z

### 3. Recent Activity

- Last synced specs (sort by `ado.syncedAt`)
- Recently updated work items

### 4. Stale Items

- Specs modified after `ado.syncedAt` (need resync)
- Work items not updated in 30+ days

## Query Examples

### Find all OpenSpec work items in the configured team backlog

```
mcp__ado-work-items__wit_query_by_wiql
- wiql: "SELECT [System.Id],[System.Title],[System.State],[System.CreatedDate] FROM WorkItems WHERE [System.Tags] CONTAINS 'openspec' AND [System.AreaPath] UNDER '<ado.areaPath>' ORDER BY [System.CreatedDate] DESC"
- project: <project-name>
```

### Find all OpenSpec work items (all teams)

```
mcp__ado-work-items__wit_query_by_wiql
- wiql: "SELECT [System.Id],[System.Title],[System.State],[System.CreatedDate] FROM WorkItems WHERE [System.Tags] CONTAINS 'openspec' ORDER BY [System.CreatedDate] DESC"
- project: <project-name>
```

### Find work items assigned to me

```
mcp__ado-work-items__wit_my_work_items
- type: "assignedtome"
- includeCompleted: false
```

### Find items in current sprint

```
mcp__ado-work-items__wit_get_work_items_for_iteration
- iterationPath: "Project\\Sprint 5"
```

## Report Template

```markdown
# ADO Sync Status Report

**Generated:** {timestamp}

## Summary
- Total specs: X
- Synced to ADO: Y (Z%)
- Work items created: Y

## By State
- To Do: X
- In Progress: Y
- Done: Z

## Recent Activity
- Last synced: {spec-name} at {timestamp}
- Recently updated: {work-item-title} (#{id})

## Action Items
- [ ] X specs need initial sync
- [ ] Y specs modified since last sync
- [ ] Z orphaned work items to review
```

## MCP Tools Used

- `mcp__ado-work-items__core_list_projects` — List available projects
- `mcp__ado-work-items__core_list_project_teams` — List teams to select target backlog
- `mcp__ado-work-items__wit_get_work_item` — Get work item details
- `mcp__ado-work-items__wit_get_work_items_batch_by_ids` — Fetch multiple work items
- `mcp__ado-work-items__wit_query_by_wiql` — Search via WIQL query
- `mcp__ado-work-items__wit_my_work_items` — Get items assigned to me
- `mcp__ado-work-items__wit_update_work_item` — Update a work item
- `mcp__ado-work-items__wit_update_work_items_batch` — Batch update work items
- `mcp__ado-work-items__wit_add_work_item_comment` — Add comment
- `mcp__ado-work-items__wit_work_items_link` — Link work items
- `mcp__ado-work-items__wit_add_child_work_items` — Create child tasks
- `mcp__ado-work-items__wit_get_work_items_for_iteration` — Items by sprint

## Related Skills

- `/azdo-sync-specs` — Sync specs to work items
- `/azdo-setup` — Configure ADO integration
