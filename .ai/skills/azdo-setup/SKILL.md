---
name: azdo-setup
description: 'Configure Azure DevOps integration for OpenSpec. Use when setting up ADO connection, managing MCP authentication, or troubleshooting connectivity.'
argument-hint: 'Organization and project details'
---

# Azure DevOps - Setup & Configuration

Configure Azure DevOps integration for OpenSpec projects using the `ado-work-items` MCP server.

## When to Use

- Initial ADO setup for OpenSpec project
- Update organization or project settings
- Troubleshoot MCP authentication issues
- Validate ADO connection
- Select or change the active project

## Authentication

Authentication is handled by the **`ado-work-items` MCP server** — no PAT token environment variable is required.

When you call any `mcp__ado-work-items__*` tool, the MCP server manages credentials interactively. If it needs to prompt for authentication, follow the on-screen instructions.

## Procedure

### 1. Verify MCP Server Availability

Confirm the MCP server is responding by listing projects:

```
mcp__ado-work-items__core_list_projects
```

**If this call fails:**
- Confirm the `ado-work-items` MCP server is enabled in your Claude Code MCP settings
- Check that your Azure DevOps organization URL is configured in the MCP server settings
- Restart the MCP server if needed: Claude Code → Settings → MCP Servers

### 2. Select Project

List all available projects and ask the user which one to use:

```
mcp__ado-work-items__core_list_projects
```

Show the project list and ask the user to confirm the target project.

### 3. Select Target Backlog / Team

List the teams in the selected project and ask the user which team's backlog work items should be synced to:

```
mcp__ado-work-items__core_list_project_teams
- project: <project-name>
```

Display the team list and ask the user to select one. The selected team determines:
- **`ado.team`** — the team name (used for backlog queries)
- **`ado.areaPath`** — the area path assigned to new work items so they appear on the correct team backlog

If the user selects a team, derive the area path as `<project>\<team-name>` unless the team's area differs — in that case, ask the user to confirm the area path.

Save both values to `openspec/config.yaml`.

### 4. Save Configuration (Optional)

Save organization, project, and team backlog to `openspec/config.yaml` for skills to reuse:

```yaml
# Azure DevOps integration
ado:
  organization: your-org-name
  project: your-project-name
  team: your-team-name          # team whose backlog receives synced work items
  areaPath: your-project\your-team-name  # area path set on all created work items

  # Optional: customize work item mapping
  workItemType: Task  # or "User Story", "Bug", etc.

  # Optional: custom field mappings
  fieldMappings:
    title: System.Title
    description: System.Description
    acceptanceCriteria: Microsoft.VSTS.Common.AcceptanceCriteria
```

**Do not add a `pat:` field.** Authentication is managed by the MCP server.

### 5. Validate Project Access

Verify the selected project is accessible:

```
mcp__ado-work-items__core_list_projects
- projectNameFilter: <project-name>
```

Should return the configured project.

### 6. Test Work Item Access

Confirm read/write access by listing work item types:

```
mcp__ado-work-items__wit_get_work_item_type
- project: <project-name>
- workItemType: Task
```

## Configuration Options

### Work Item Type

Default: `Task`

Common alternatives:
- `User Story` — Agile/Scrum projects
- `Bug` — Defect tracking
- `Epic` — High-level features
- `Feature` — Larger work items

### Custom Field Mappings

Map spec sections to custom fields:

```yaml
ado:
  fieldMappings:
    title: System.Title
    description: System.Description
    acceptanceCriteria: Microsoft.VSTS.Common.AcceptanceCriteria
    priority: Microsoft.VSTS.Common.Priority
    tags: System.Tags
    myCustomField: Custom.MyField
```

## Troubleshooting

### "MCP server not available"

- Open Claude Code Settings → MCP Servers
- Verify `ado-work-items` server is listed and enabled
- Check the server configuration includes your ADO organization URL
- Try restarting: disable and re-enable the server

### "Project not found"

1. List all projects: `mcp__ado-work-items__core_list_projects`
2. Copy exact project name from the list (case-sensitive)
3. Verify your ADO user account has access to the project

### "Permission denied"

- Confirm your ADO account has Contributor access to the project
- Re-authenticate via the MCP server if credentials have expired
- Contact your ADO project admin to grant access

## Example Complete Configuration

```yaml
schema: spec-driven

# Azure DevOps integration
ado:
  organization: contoso
  project: ProductBacklog
  team: Platform Team                  # team whose backlog receives synced work items
  areaPath: ProductBacklog\Platform Team  # area path set on all created work items
  workItemType: User Story

  # Tags added to all work items
  defaultTags:
    - openspec
    - automated

  # Optional: iteration path
  iterationPath: ProductBacklog\Sprint 1

# Project context
context: |
  Tech stack: TypeScript, Node.js, Azure
  Domain: Product backlog management
  Process: Scrum with 2-week sprints
```

## Next Steps

After successful setup:
1. Sync a spec: `/azdo-sync-specs`
2. Query work items: `/azdo-query`
3. Sync documentation to wiki: `/azdo-sync-wiki`
