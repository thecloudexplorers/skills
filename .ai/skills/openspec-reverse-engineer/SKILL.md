---
name: openspec-reverse-engineer
description: 'Reverse engineer existing codebases into OpenSpec format. Use when analyzing repos to extract features, capabilities, and AI skills into structured specs with requirements and scenarios.'
argument-hint: 'path or feature to analyze'
---

# OpenSpec Reverse Engineering

Analyze existing codebases and convert discovered features into OpenSpec specification format, including proper requirement structure, scenarios, and optional Azure DevOps work item mapping.

## When to Use

- Converting legacy code to spec-driven development
- Documenting existing features in OpenSpec format
- Creating specs from AI skills (SKILL.md files)
- Establishing ADO work item hierarchy from codebase features

## Discovery Phase

### 1. Identify Scope

Ask the user:
- Analyze entire repo or specific directory?
- Looking for specific feature areas?
- Include AI skills (`.github/skills/`, `.agents/skills/`, `.claude/skills/`)?

### 2. Explore Codebase

Use semantic search and file exploration to discover:

**Code Features:**
- Key modules and their responsibilities
- Public APIs and interfaces
- Main workflows and use cases
- Configuration and setup requirements

**AI Skills:**
- Scan for `SKILL.md` files in standard locations
- Extract workflow steps and procedures
- Identify tool usage and integrations
- Note any bundled scripts or resources

## Extraction Phase

### 3. Extract Requirements

For each discovered feature or skill, create structured requirements:

**Format:**
```markdown
### Requirement: [Concise Title]
As a [role], I want to [action] so that [benefit].

The system SHALL [technical requirement statement].
```

**Mapping Guidelines:**
- Feature/Skill → OpenSpec Requirement
- User workflows → User Stories (As a...)
- Technical capabilities → SHALL statements
- Setup/config → Configuration Requirements

### 4. Define Scenarios

Convert workflows and use cases into testable scenarios:

**Format:**
```markdown
#### Scenario: [Clear scenario name]
- **WHEN** [precondition or trigger]
- **THEN** [expected outcome]
```

**Sources:**
- Test cases → Scenarios
- Example usage → Success scenarios
- Error handling → Failure scenarios
- Edge cases → Boundary scenarios

## Spec Creation Phase

### 5. Generate OpenSpec Files

Create spec files following OpenSpec structure:

**File Location:** `openspec/changes/[feature-name]/specs/[spec-name]/spec.md`

**Template:**
```markdown
---
# Optional: ADO work item tracking
ado:
  feature:
    id: [number]
    title: "[SKILL] [Feature Name]"
  userStories:
    - id: [number]
      title: "[Short Title]"
      tasks:
        - id: [number]
          title: "[Task Title]"
---

## ADDED Requirements

### Requirement: [Title]
As a [role], I want to [action] so that [benefit].

The system SHALL [requirement statement].

#### Scenario: [Name]
- **WHEN** [condition]
- **THEN** [outcome]
```

### 6. Map to Work Items (Optional)

If Azure DevOps integration is configured:

**Hierarchy:**
- Feature: Top-level capability (prefix with [SKILL] for AI skills)
- User Story: Each requirement with user story format
- Tasks: Individual scenarios or implementation steps

**Field Mapping:**
- Title: Short, descriptive name
- Description: Full user story + SHALL statement (HTML format)
- Acceptance Criteria: Scenario conditions (HTML list format)

## Quality Checklist

Before finalizing specs:

- [ ] Each requirement has clear user story format
- [ ] SHALL statements are unambiguous
- [ ] Scenarios are testable (WHEN/THEN format)
- [ ] Titles are concise and meaningful
- [ ] ADO metadata is complete (if using)
- [ ] Files follow OpenSpec directory structure

## Examples

**From Code:**
```python
def authenticate(token: str) -> Session:
    """Authenticate with API using bearer token"""
```

**To Spec:**
```markdown
### Requirement: API Authentication
As a developer, I want to authenticate with the API using tokens so that I can access protected resources securely.

The system SHALL authenticate users via bearer tokens.

#### Scenario: Successful authentication
- **WHEN** a valid bearer token is provided
- **THEN** the system returns an authenticated session
```

**From AI Skill:**
```markdown
---
name: azdo-sync-specs
description: 'Sync OpenSpec specs to Azure DevOps work items'
---
```

**To Spec:**
```markdown
### Requirement: Sync Specs to Azure DevOps
As a user, I want to sync specs to ADO work items so that my specifications are tracked in the backlog.

The system SHALL create or update Azure DevOps work items from OpenSpec files.

#### Scenario: Create work item from spec
- **WHEN** a spec is synced to ADO for the first time
- **THEN** the system creates a new work item with mapped fields
```

## Tips

**Progressive Approach:**
1. Start with major features/modules
2. Group related capabilities into single specs
3. Extract common patterns first
4. Add detail iteratively

**Avoid:**
- Creating one spec per function (too granular)
- Copying code comments verbatim (transform to requirements)
- Missing the "why" (always include user story context)
- Vague scenarios (be specific about WHEN and THEN)

## Next Steps

After creating specs:
1. Review with stakeholders for accuracy
2. Sync to Azure DevOps (if configured)
3. Use specs to guide refactoring or documentation
4. Keep specs updated as code evolves
