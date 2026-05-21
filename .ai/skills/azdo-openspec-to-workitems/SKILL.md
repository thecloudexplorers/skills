---
name: openspec-to-azure-devops-items
description: Convert OpenSpec specs, requirements, proposals, tasks, and validation items into Azure DevOps work items.
---

# OpenSpec to Azure DevOps Work Items Skill

## Purpose

Convert OpenSpec items into Azure DevOps work items using a clear and consistent mapping.

This skill transforms OpenSpec-based specification work into an Azure DevOps backlog structure with:

- Epics
- Features
- User Stories
- Tasks
- Test Cases
- Notes / documentation items

---

## Input

The user may provide OpenSpec content such as:

- Spec
- Requirement
- Proposal
- Design
- Spec delta
- Task
- Acceptance criteria
- Constraint
- Assumption
- Validation rule
- Scenario

The input may be:

- Raw Markdown
- OpenSpec folder content
- A summary
- A feature idea
- A change proposal
- A list of requirements

---

## Output Structure

Return Azure DevOps work items using this hierarchy:

    Epic
    └── Feature
        └── User Story
            ├── Task
            ├── Task
            └── Test Case

---

## Mapping Rules

| OpenSpec item         | Azure DevOps equivalent                     | Notes                                           |
| --------------------- | ------------------------------------------- | ----------------------------------------------- |
| Project goal          | Epic                                        | Use for broad business or platform objectives   |
| Spec                  | Feature                                     | Use when the spec describes a system capability |
| Capability            | Feature                                     | Prefer Feature unless the scope is very large   |
| Requirement           | User Story                                  | Convert each requirement into one user story    |
| Scenario              | Acceptance Criteria or Test Case            | Use Given/When/Then when possible               |
| Acceptance Criteria   | Acceptance Criteria                         | Add under the related User Story                |
| Constraint            | Acceptance Criteria or Notes                | Use Acceptance Criteria if it must be validated |
| Assumption            | Description or Notes                        | Create a Task only if it must be verified       |
| Proposal              | Feature or Change Request                   | Use Feature if it delivers value                |
| Design                | Feature description or linked documentation | Summarize key delivery decisions                |
| Spec Delta - ADDED    | New User Story / Acceptance Criteria        | Create new backlog items                        |
| Spec Delta - MODIFIED | Updated User Story / Acceptance Criteria    | Mark as modification                            |
| Spec Delta - REMOVED  | Removal Story / Cleanup Task                | Include migration or deprecation impact         |
| Task                  | Task                                        | Technical implementation work                   |
| Validation            | Test Case                                   | Convert into verification steps                 |

---

## Conversion Rules

### 1. Identify the hierarchy

Use this structure:

- Epic for broad initiatives
- Feature for OpenSpec specs, capabilities, or proposals
- User Story for each requirement
- Task for implementation steps
- Test Case for validation rules or scenarios

Do not create unnecessary Epics if the scope is small.

---

### 2. Convert Specs to Features

Format:

    ## Feature: <Feature Name>

    **Source:** OpenSpec Spec
    **Area:** <Area or capability name>
    **State:** New / Modified / Existing

    ### Description

    <Short explanation of the capability.>

    ### Business / Technical Value

    <Why this feature matters.>

    ### Scope

    Included:

    - <Included item>

    Excluded:

    - <Excluded item>

    ### Linked User Stories

    - <User Story title>

---

### 3. Convert Requirements to User Stories

Each OpenSpec requirement becomes one Azure DevOps User Story.

Format:

    ## User Story: <Story Title>

    **Source:** OpenSpec Requirement
    **Parent Feature:** <Feature Name>

    ### User Story

    As a <user/system/role>,
    I want <capability/behavior>,
    so that <value/outcome>.

    ### Description

    <Explain the requirement in plain language.>

    ### Acceptance Criteria

    - [ ] <Condition that must be true>
    - [ ] <Condition that must be true>

    ### Definition of Done

    - [ ] <Condition that must be true>
    - [ ] <Condition that must be true>

    ### Constraints

    - <Constraint, if any>

    ### Assumptions

    - <Assumption, if any>

    ### Notes

    - <Relevant design or implementation notes>

For system-focused requirements, use:

    As the system,
    I want to <behavior>,
    so that <technical outcome>.

---

### 4. Convert Scenarios to Acceptance Criteria

Use Given/When/Then format where possible.

Example:

    ### Acceptance Criteria

    - [ ] Given <context>, when <action>, then <expected result>.
    - [ ] Given <context>, when <failure condition>, then <expected handling>.

---

### 5. Convert Tasks to Azure DevOps Tasks

Each implementation task becomes an Azure DevOps Task.

Format:

    ## Task: <Task Title>

    **Source:** OpenSpec Task
    **Parent User Story:** <User Story Title>

    ### Description

    <What needs to be done.>

    ### Checklist

    - [ ] <Implementation step>
    - [ ] <Implementation step>

    ### Done When

    - [ ] <Concrete completion condition>

Good task examples:

- Create Bicep module for diagnostic settings
- Add validation for required tags
- Update pipeline template parameters
- Add unit tests for policy assignment logic

Avoid vague tasks such as:

- Improve system
- Fix everything
- Implement feature

---

### 6. Convert Validation to Test Cases

Validation rules, examples, or scenarios should become Test Cases.

Format:

    ## Test Case: <Test Case Title>

    **Source:** OpenSpec Validation / Scenario
    **Related User Story:** <User Story Title>

    ### Preconditions

    - <Required setup>

    ### Test Steps

    1. <Action>
    2. <Action>
    3. <Action>

    ### Expected Result

    <Expected behavior or outcome>

---

## Output Format

Always return the result in this order:

    # Azure DevOps Work Item Conversion

    ## Summary

    <Short summary of what was converted.>

    ## Proposed Work Item Hierarchy

    - Epic: <name>
      - Feature: <name>
        - User Story: <name>
          - Task: <name>
          - Test Case: <name>

    ## Work Items

    <All generated work items here>

    ## Traceability Matrix

    | OpenSpec Item | Azure DevOps Item | Type | Notes |
    |---|---|---|---|
    | <OpenSpec item> | <ADO item> | <Type> | <Notes> |

    ## Gaps / Questions

    - <Missing information>
    - <Ambiguity>

---

## Traceability Rules

Always include a traceability matrix.

The matrix must show:

- Original OpenSpec item
- Generated Azure DevOps item
- Work item type
- Notes about assumptions or changes

Example:

| OpenSpec Item               | Azure DevOps Item                  | Type       | Notes                        |
| --------------------------- | ---------------------------------- | ---------- | ---------------------------- |
| auth-session spec           | Enable Remember Me Authentication  | Feature    | Created from capability spec |
| Requirement: 30-day session | Remember Me 30-Day Session         | User Story | Converted from requirement   |
| Scenario: expired token     | Expired Remember Me Token Handling | Test Case  | Converted from scenario      |

---

## Naming Rules

Use clear Azure DevOps-style names.

Prefer:

- Verb + outcome
- Capability + behavior
- Business/system value

Examples:

- Enable Remember Me Authentication
- Validate Required Azure Tags
- Publish Deployment Artifact
- Enforce Subscription Naming Convention

Avoid:

- OpenSpec Requirement 1
- Auth Stuff
- Fix Login
- Misc Tasks

---

## Rules for Constraints and Assumptions

### Constraints

If a constraint affects delivery or validation, include it in Acceptance Criteria.

Example:

    ### Acceptance Criteria

    - [ ] The session token must expire after 30 days.
    - [ ] The solution must not store plain-text tokens.

### Assumptions

If an assumption is informational, place it under Assumptions.

Example:

    ### Assumptions

    - The user identity provider already supports refresh tokens.

If an assumption must be verified, create a Task.

Example:

    ## Task: Verify identity provider refresh token support

---

## Handling OpenSpec Deltas

### ADDED

Create new Azure DevOps items.

    Change Type: Added
    Action: Create new Feature/User Story/Task/Test Case

### MODIFIED

Update or create a replacement item.

    Change Type: Modified
    Action: Update existing work item or create a new change story

### REMOVED

Create a removal or cleanup story/task.

    Change Type: Removed
    Action: Create removal story, cleanup task, or deprecation test case

---

## Quality Checklist

Before returning the final output, verify:

- [ ] Every requirement became a User Story.
- [ ] Every task is actionable.
- [ ] Every validation rule became Acceptance Criteria or Test Case.
- [ ] Every assumption is either documented or converted into a verification task.
- [ ] Every constraint is captured.
- [ ] The hierarchy is clear.
- [ ] The traceability matrix is complete.
- [ ] No OpenSpec item was silently ignored.

---

## Behavior Rules

When information is missing:

- Do not stop.
- Make a reasonable assumption.
- Mark it clearly under `Gaps / Questions`.

When the input is too large:

- Summarize the hierarchy first.
- Convert the highest-value items.
- Mention what was not converted.

When the user asks for Azure DevOps-ready output:

- Use concise titles.
- Use copy-pasteable Markdown.
- Avoid long explanations.
- Keep the structure close to Azure DevOps work item fields.
