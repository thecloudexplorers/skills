---
name: azdo-automate
description: "Automate Azure DevOps workflows for OpenSpec. Use for bulk operations, scheduled syncs, or CI/CD integration."
argument-hint: "Automation task or workflow type"
---

# Azure DevOps - Automation

Automate repetitive Azure DevOps tasks and create workflows for OpenSpec spec management.

## When to Use

- Bulk sync all specs to ADO
- Automated resync on spec changes
- CI/CD pipeline integration
- Scheduled status reports
- Batch work item updates

## Prerequisites

- The `ado-work-items` MCP server is configured and enabled in Claude Code

For interactive use, authentication is handled automatically by the MCP server.
For CI/CD pipelines, configure a service principal or PAT in the pipeline secrets and inject as required by the MCP server configuration.

## Automation Workflows

### 1. Bulk Sync All Specs

Sync all specs in the workspace to Azure DevOps:

**Process:**

1. Find all spec.md files in workspace
2. Resolve target project, team, and area path from `openspec/config.yaml`:
   - If `ado.project` is missing → call `mcp__ado-work-items__core_list_projects` and ask the user to select
   - If `ado.team` or `ado.areaPath` is missing → call `mcp__ado-work-items__core_list_project_teams` and ask: _"Which team's backlog should these work items be synced to?"_
   - Persist any newly selected values to `openspec/config.yaml` before proceeding
3. For each spec:
   - Read spec content and check frontmatter for existing `ado.hierarchy` IDs
   - If no hierarchy: apply `azdo-openspec-to-workitems` to get the work item structure, then create all items (Epic → Feature → User Story → Task → Test Case) and link them
   - If hierarchy exists: check if spec was modified since `ado.syncedAt`; if yes, re-run `azdo-openspec-to-workitems` and update the affected work items
   - If unchanged: skip
4. Write IDs back to each spec's frontmatter
5. Generate summary report

The `azdo-openspec-to-workitems` skill handles all content mapping and hierarchy decisions. The automation skill only orchestrates file discovery, calls to that skill, and MCP execution.

**Efficiency tips:**

- Process one change folder at a time (group specs by change)
- Cache project configuration within the session
- Rate limit to avoid API throttling (200 req/min)
- Use `mcp__ado-work-items__wit_add_child_work_items` to create children in bulk under a known parent

### 2. Watch Mode - Continuous Sync

Monitor specs for changes and auto-sync:

**Setup:**

1. Start file watcher on spec directories
2. On file change event:
   - Debounce (wait 5 seconds for multiple edits)
   - Read changed spec
   - Update corresponding work item
   - Log sync activity

**Implementation:**

```powershell
# File watcher script
$watcher = New-Object System.IO.FileSystemWatcher
$watcher.Path = "openspec/specs"
$watcher.Filter = "spec.md"
$watcher.IncludeSubdirectories = $true
$watcher.EnableRaisingEvents = $true

$action = {
    $path = $Event.SourceEventArgs.FullPath
    Write-Host "Spec changed: $path"
    # Trigger sync via MCP tools
}

Register-ObjectEvent $watcher "Changed" -Action $action
```

### 3. Status Report Automation

Generate periodic sync status reports:

**Daily/Weekly report:**

1. Query all work items with "openspec" tag:
   ```
   mcp__ado-work-items__wit_query_by_wiql
   - wiql: "SELECT [System.Id],[System.Title],[System.State] FROM WorkItems WHERE [System.Tags] CONTAINS 'openspec'"
   ```
2. Group by state (To Do, In Progress, Done)
3. Calculate metrics: total synced specs, completion percentage, items modified this week
4. Format as markdown report

**Report schedule (Azure Pipeline):**

```yaml
name: ADO Sync Status Report
schedules:
  - cron: "0 9 * * 1" # Every Monday at 9 AM UTC
    branches:
      include:
        - main

jobs:
  - job: StatusReport
    steps:
      - script: |
          # Run azdo-query skill to collect stats and post report
        displayName: Generate status report
```

### 4. Pre-Commit Hook

Sync specs automatically before committing:

```bash
# .git/hooks/pre-commit
#!/bin/bash

# Find modified spec files
SPECS=$(git diff --cached --name-only | grep "spec.md$")

if [ -n "$SPECS" ]; then
  echo "Modified specs detected, syncing to ADO..."
  for spec in $SPECS; do
    echo "  -> $spec"
    # Trigger sync for each modified spec via MCP tools
  done
fi
```

### 5. CI/CD Integration

Integrate ADO sync into Azure Pipelines:

```yaml
# azure-pipelines-sync-specs.yml
trigger:
  paths:
    include:
      - openspec/specs/**/*.md
      - openspec/changes/**/*.md

pool:
  vmImage: ubuntu-latest

steps:
  - checkout: self
    persistCredentials: true

  - task: AzureCLI@2
    displayName: Sync Specs to ADO
    inputs:
      azureSubscription: <service-connection>
      scriptType: bash
      scriptLocation: inlineScript
      inlineScript: |
        # Use ADO REST API or MCP server to sync specs
        # Configure MCP server with service principal credentials

  - script: |
      git config user.name "Azure Pipelines"
      git config user.email "pipeline@dev.azure.com"
      git add openspec/**/*.md
      git commit -m "Update ADO sync metadata [skip ci]" || true
      git push
    displayName: Commit metadata updates
```

### 6. Batch State Updates

Update multiple work items based on spec status:

**Use cases:**

- Mark all completed specs' work items as Done
- Set priority based on spec tags
- Assign work items to team members
- Apply bulk tags

**Implementation:**

```
# Find all synced spec work item IDs from spec frontmatter
# Then batch update:
mcp__ado-work-items__wit_update_work_items_batch
- updates:
  - id: 1
    path: "/fields/System.State"
    value: "Done"
  - id: 2
    path: "/fields/System.State"
    value: "Done"
```

### 7. Spec-to-Task Breakdown

Automatically create a full work item hierarchy from a spec:

**Process:**

1. Read spec file
2. Apply `azdo-openspec-to-workitems` — it extracts all requirements, scenarios, tasks, and validation rules and maps them to the correct ADO types (User Story, Task, Test Case)
3. Take the resulting hierarchy and execute it via MCP:
   - Create Feature (or Epic if needed)
   - Create User Stories under the Feature
   - Create Tasks and Test Cases under each User Story using `mcp__ado-work-items__wit_add_child_work_items`
4. Link all items using `mcp__ado-work-items__wit_work_items_link` (Hierarchy-Forward)

**Example output from `azdo-openspec-to-workitems`:**

```
Feature: Cloud Governance Foundation
  └─ User Story: Enforce subscription naming convention
      ├─ Task: Implement naming policy Bicep module
      ├─ Task: Add pipeline validation step
      └─ Test Case: Validate naming compliance
  └─ User Story: Apply required resource tags
      ├─ Task: Create tag policy assignment
      └─ Test Case: Verify tag enforcement
```

## Bulk Operations

### Sync by Pattern

Sync specs matching a pattern:

```bash
# Find all network-related specs
find openspec/changes/enterprise-network-governance -name "spec.md"
# For each: read, create/update work item via MCP
```

### Update All Tags

Add or update tags on all synced work items:

```
1. Query all work items with "openspec" tag via mcp__ado-work-items__wit_query_by_wiql
2. For each work item:
   - Add new tag (e.g., "sprint-5", "priority-high")
   - Use mcp__ado-work-items__wit_update_work_item
```

### Archive Completed Work

Close work items for archived specs:

```
1. List specs in openspec/changes/archive/
2. Extract work item IDs from frontmatter
3. Batch update state to "Closed" via mcp__ado-work-items__wit_update_work_items_batch
4. Add comment: "Spec archived in OpenSpec" via mcp__ado-work-items__wit_add_work_item_comment
```

## Performance Optimization

### Rate Limiting

Respect Azure DevOps API rate limits:

- Max ~200 requests per minute
- Use batch operations (`wit_update_work_items_batch`, `wit_get_work_items_batch_by_ids`) to reduce call count
- Add small delays between bulk operations if hitting limits

### Caching

Within a session, cache:

- Project configuration (org name, project name, work item types)
- Wiki identifiers
- Recently fetched work item IDs

## Error Handling & Retry

Implement retry logic for transient failures:

1. On `429 Too Many Requests`: wait and retry with backoff
2. On `503 Service Unavailable`: retry after 30 seconds
3. On `401 Unauthorized`: prompt user to re-authenticate via MCP server settings
4. On `404 Not Found`: log and skip (work item may have been deleted)

## Monitoring & Logging

### Log Sync Operations

Track sync activities in a local log file:

```json
{
  "timestamp": "2026-05-21T10:00:00Z",
  "operation": "sync",
  "spec": "cloud-governance-foundation",
  "work_item_id": 42,
  "action": "updated",
  "status": "success",
  "duration_ms": 850
}
```

### Metrics to Track

- Total syncs per session
- Success vs failure rate
- Specs with stale work items (modified after `ado.syncedAt`)

## MCP Tools Used

- `mcp__ado-work-items__core_list_projects` — Resolve target project
- `mcp__ado-work-items__core_list_project_teams` — List teams to select target backlog
- `mcp__ado-work-items__wit_create_work_item` — Create new work item
- `mcp__ado-work-items__wit_update_work_item` — Update single work item
- `mcp__ado-work-items__wit_update_work_items_batch` — Batch update work items
- `mcp__ado-work-items__wit_get_work_items_batch_by_ids` — Fetch multiple work items
- `mcp__ado-work-items__wit_query_by_wiql` — Search via WIQL
- `mcp__ado-work-items__wit_add_child_work_items` — Create child tasks
- `mcp__ado-work-items__wit_add_work_item_comment` — Add comment

## Related Skills

- `azdo-openspec-to-workitems` — Transforms spec content into the ADO work item hierarchy used by bulk sync
- `/azdo-sync-specs` — Manual sync for a single spec or change
- `/azdo-query` — Query work items
- `/azdo-setup` — Configure integration
