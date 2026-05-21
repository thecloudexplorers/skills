# Azure DevOps Skills for OpenSpec

A collection of skills for integrating OpenSpec specifications with Azure DevOps work items using the `ado-work-items` MCP server.

## Authentication

All skills use the **`ado-work-items` MCP server** for authentication — no PAT token environment variable is required. The MCP server handles credentials interactively when needed.

To configure: Claude Code → Settings → MCP Servers → `ado-work-items`

## Skills in this Group

### [azdo-sync-specs](../azdo-sync-specs/)
Sync OpenSpec specs to Azure DevOps work items using MCP ADO integration.

**Use when:**
- Creating ADO work items from specs
- Updating existing work items
- Testing ADO integration

**Key features:**
- Reads spec requirements and scenarios
- Creates/updates work items via MCP tools
- Updates spec metadata with work item IDs

### [azdo-setup](../azdo-setup/)
Configure Azure DevOps integration for OpenSpec projects.

**Use when:**
- Initial ADO setup
- Updating organization/project settings
- Troubleshooting MCP authentication
- Selecting the active project

**Key features:**
- Guides through `openspec/config.yaml` setup (no PAT field)
- MCP server configuration instructions
- Connection validation via `mcp__ado-work-items__core_list_projects`

### [azdo-query](../azdo-query/)
Query and manage Azure DevOps work items related to OpenSpec.

**Use when:**
- Searching for work items
- Checking sync status
- Generating reports
- Managing backlog items

**Key features:**
- Search work items by spec
- Find orphaned or missing items
- Batch operations
- Status reporting

### [azdo-automate](../azdo-automate/)
Automate Azure DevOps workflows for OpenSpec.

**Use when:**
- Bulk syncing multiple specs
- CI/CD integration
- Scheduled operations
- Automated reporting

**Key features:**
- Bulk sync workflows
- CI/CD pipeline examples (Azure Pipelines)
- Rate limiting and error handling
- Monitoring and logging

### [azdo-sync-wiki](../azdo-sync-wiki/)
Sync OpenSpec documentation to Azure DevOps Wiki.

**Use when:**
- Publishing proposals and design docs to the team wiki
- Keeping ADO Wiki in sync with OpenSpec changes

## Quick Start

1. **Setup ADO Integration**
   ```
   /azdo-setup
   ```
   Confirm MCP server is working and select your project.

2. **Sync Your First Spec**
   ```
   sync the cloud-governance-foundation specs to azure devops
   ```
   or explicitly invoke:
   ```
   /azdo-sync-specs cloud-governance-foundation
   ```

3. **Query Work Items**
   ```
   /azdo-query openspec
   ```

4. **Sync Docs to Wiki**
   ```
   /azdo-sync-wiki cloud-governance-foundation
   ```

5. **Automate** (optional)
   ```
   /azdo-automate bulk-sync
   ```

## Prerequisites

- Azure DevOps organization and project
- `ado-work-items` MCP server configured in Claude Code settings
- MCP server account has Contributor access to the target project

## Configuration Example

```yaml
# openspec/config.yaml
schema: spec-driven

# Azure DevOps integration
ado:
  organization: your-org
  project: your-project
  team: Platform Team                   # team whose backlog receives synced work items
  areaPath: your-project\Platform Team  # area path set on all created work items
  workItemType: Task
```

No `pat:` field — authentication is managed by the MCP server.

`team` and `areaPath` are written automatically the first time you run any azdo skill and select a team. To change the target backlog, update these fields in `openspec/config.yaml` or run `/azdo-setup`.

## How Sync Works

Syncing a spec is a two-phase process:

**Phase 1 — Transform** (`azdo-openspec-to-workitems` skill)
Reads spec content and produces the complete ADO work item hierarchy: Epic → Feature → User Story → Task → Test Case, with descriptions, acceptance criteria, and a traceability matrix.

**Phase 2 — Execute** (`azdo-sync-specs` or `azdo-automate` skill)
Takes the hierarchy from phase 1 and creates or updates real work items in ADO via MCP tools, links parent-child relationships, and writes IDs back to spec frontmatter.

You can run both phases together (sync calls transform internally), or separately — useful when you want to review the proposed hierarchy before writing to ADO.

## Common Workflows

### Sync a Single Spec
1. Resolve project from config or interactively
2. Read spec content
3. Apply `azdo-openspec-to-workitems` → get work item hierarchy
4. Review hierarchy with user (last checkpoint before writing to ADO)
5. Create Epic/Feature/User Story/Task/Test Case via MCP tools, parent-first
6. Link parent-child relationships
7. Write all created IDs to spec frontmatter under `ado.hierarchy`

### Bulk Sync All Specs
1. Find all spec.md files
2. For each spec: check frontmatter for existing `ado.hierarchy` IDs
3. New specs: run transform → create full hierarchy → write IDs
4. Modified specs (changed after `ado.syncedAt`): re-run transform → update changed items
5. Generate summary report

### Review Before Syncing
1. Run `azdo-openspec-to-workitems` on spec content
2. Review the proposed hierarchy, descriptions, and traceability matrix
3. Adjust spec if needed, then re-run
4. Hand output to `azdo-sync-specs` to execute

### Check Sync Status
1. Query work items with "openspec" tag via WIQL
2. Compare with spec files in workspace
3. Identify:
   - Specs without `ado.hierarchy` frontmatter (not yet synced)
   - Work items without matching specs (orphaned)
   - Specs modified after `ado.syncedAt` (need resync)

## MCP Tools Reference

| Tool | Purpose |
|------|---------|
| `mcp__ado-work-items__core_list_projects` | List available projects |
| `mcp__ado-work-items__wit_create_work_item` | Create new work items |
| `mcp__ado-work-items__wit_update_work_item` | Update existing work items |
| `mcp__ado-work-items__wit_get_work_item` | Retrieve work item details |
| `mcp__ado-work-items__wit_get_work_items_batch_by_ids` | Fetch multiple work items |
| `mcp__ado-work-items__wit_update_work_items_batch` | Batch updates |
| `mcp__ado-work-items__wit_query_by_wiql` | Search via WIQL |
| `mcp__ado-work-items__wit_my_work_items` | Get assigned items |
| `mcp__ado-work-items__wit_add_child_work_items` | Create child tasks |
| `mcp__ado-work-items__wit_add_work_item_comment` | Add comments |
| `mcp__ado-work-items__wiki_list_wikis` | List wikis |
| `mcp__ado-work-items__wiki_create_or_update_page` | Create/update wiki page |

## Field Mapping

Default mapping from spec to work item:

| Spec Element | ADO Field | Format |
|--------------|-----------|--------|
| Spec name/title | System.Title | Plain text |
| Requirements + scenarios | System.Description | HTML |
| Validation points | Microsoft.VSTS.Common.AcceptanceCriteria | HTML checklist |
| Spec tags | System.Tags | Semicolon-separated |

## Troubleshooting

### MCP Server Not Available
- Open Claude Code Settings → MCP Servers
- Verify `ado-work-items` is listed and enabled
- Check it's configured with your ADO organization URL
- Restart the MCP server

### Project Not Found
- Run `mcp__ado-work-items__core_list_projects` to list available projects
- Verify exact project name (case-sensitive)
- Confirm MCP server account has access to the project

### Work Item Creation Failed
- Check work item type exists in project
- Try with minimal fields (just `System.Title`)
- Verify field names match ADO schema

### Permission Denied
- Confirm MCP server account has Contributor access
- Contact project admin to grant access

## Related OpenSpec Skills

- `azdo-openspec-to-workitems` — Transforms spec content into the ADO work item hierarchy (Phase 1 of sync)
- `openspec-apply-change` — Apply changes to codebase
- `openspec-archive-change` — Archive completed changes
- `openspec-propose` — Create new change proposals
- `openspec-explore` — Explore ideas and requirements
