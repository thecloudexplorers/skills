---
name: openspec-propose-arch
description: Propose a new change from an architecture, business, and product owner viewpoint. Focuses on business capabilities, architectural decisions with alternatives, and observable outcomes — not implementation details. Use when the audience is architects, product owners, or business stakeholders rather than implementers.
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
---

Propose a new change from an architecture and business perspective.

I'll create a change with artifacts:
- proposal.md (business capabilities, value, and impact)
- design.md (architectural decisions with alternatives — not implementation)
- specs/ (observable behaviors and outcomes — not implementation details)
- tasks.md (decisions to make and outcomes to validate — not implementation steps)

When ready to implement, run /opsx:apply

---

**Input**: The user's request should include a change name (kebab-case) OR a description of what they want to build.

**Steps**

1. **If no clear input provided, ask what they want to build**

   Use the **AskUserQuestion tool** (open-ended, no preset options) to ask:
   > "What change do you want to work on? Describe what you want to build or fix."

   From their description, derive a kebab-case name (e.g., "add user authentication" → `add-user-auth`).

   **IMPORTANT**: Do NOT proceed without understanding what the user wants to build.

2. **Research the topic if external knowledge would strengthen the artifacts**

   If the change touches a well-defined domain (cloud architecture patterns, security standards, industry best practices), use available research tools (e.g., Microsoft Learn MCP, web search) to ground decisions in authoritative guidance before writing.

3. **Create the change directory**
   ```bash
   openspec new change "<name>"
   ```
   This creates a scaffolded change at `openspec/changes/<name>/` with `.openspec.yaml`.

4. **Get the artifact build order**
   ```bash
   openspec status --change "<name>" --json
   ```
   Parse the JSON to get:
   - `applyRequires`: array of artifact IDs needed before implementation
   - `artifacts`: list of all artifacts with their status and dependencies

5. **Create artifacts in sequence until apply-ready**

   Loop through artifacts in dependency order (artifacts with no pending dependencies first):

   a. **For each artifact that is `ready` (dependencies satisfied)**:
      - Get instructions:
        ```bash
        openspec instructions <artifact-id> --change "<name>" --json
        ```
      - Read any completed dependency files for context
      - Create the artifact following the content philosophy below
      - Show brief progress: "Created <artifact-id>"

   b. **Continue until all `applyRequires` artifacts are complete**

   c. **If an artifact requires user input** (unclear context):
      - Use **AskUserQuestion tool** to clarify
      - Then continue with creation

6. **Show final status**
   ```bash
   openspec status --change "<name>"
   ```

**Output**

After completing all artifacts, summarize:
- Change name and location
- List of artifacts created with brief descriptions
- What's ready: "All artifacts created! Ready for implementation."
- Prompt: "Run `/opsx:apply` or ask me to implement to start working on the tasks."

---

## Content Philosophy: WHAT and WHY, not HOW

Every artifact focuses on business capabilities, architectural intent, and observable outcomes. Implementation details — specific resource types, parameter names, tool commands, file paths — belong in the implementation phase, not here. The how is decided by the implementer based on the decisions made in these artifacts.

**Test for every sentence you write**: could an architect or product owner who doesn't know this codebase understand and validate this? If it requires knowing a specific module name or CLI flag to make sense, rewrite it.

---

## proposal.md — Business capabilities and impact

Write from the perspective of a business stakeholder and a platform consumer.

- **Why**: The business problem, risk, or opportunity being addressed. One or two sentences. Focus on consequences if not addressed.
- **What Changes**: Business capabilities added or changed. Named at the capability level, not the resource level. No file names, parameter names, or commands.
- **Capabilities**: Each capability becomes a `specs/<name>/spec.md`. Name in kebab-case. Describe what it *does for users or operators*, not what technology implements it.
- **Impact**: Who is affected, how, and at what cost. Include cost, operational burden, and any breaking changes. No code file paths.

---

## design.md — Architectural decisions with alternatives

Write as an architect briefing a decision-making audience.

- **Context**: Current architectural state and constraints. No file paths or module names — describe the architecture, not the code.
- **Goals / Non-Goals**: Capabilities this design achieves and explicitly excludes. Framed as outcomes, not features.
- **Decisions**: For each architectural decision:
  - Present **at least 3 alternatives** (4 where meaningful options exist) in a comparison table:

    ```
    | Option | Summary | Trade-offs |
    |---|---|---|
    | **A — [Chosen]** ✓ | One-line summary | Key trade-offs |
    | B — [Alternative] | One-line summary | Key trade-offs |
    | C — [Alternative] | One-line summary | Key trade-offs |
    ```

  - Follow the table with a **Rationale** sentence explaining why the chosen option wins given the specific context. If a decision is genuinely open, list all options without marking a choice, and move it to Open Questions.
  - Decisions should be architectural: topology, placement, replication strategy, observability approach — not implementation choices like which parameter to add or which module to modify.

- **Risks / Trade-offs**: Architectural and business risks with mitigations. Format: `| Risk | Mitigation |` table.
- **Migration Plan**: Phased approach at the phase level. No commands or tool invocations — describe what each phase achieves, not how it is executed.
- **Open Questions**: Architectural or business questions that must be answered before or during implementation. These drive tasks in `tasks.md`.

---

## specs/ — Observable behaviors and outcomes

Each spec defines what the system must *do* from the perspective of a user, operator, or dependent system.

- One spec file per capability listed in the proposal's Capabilities section: `specs/<capability-name>/spec.md`
- Requirements use **SHALL/MUST** for normative statements
- Scenarios use **WHEN/THEN** format and describe **observable outcomes**, not internal steps
- **Quality test**: could a QA engineer or product owner validate this scenario without reading the implementation? If yes, it is well-written.
- Do NOT mention: specific resource types, parameter names, module names, CLI commands, or tool configurations. If it names a particular Azure service or IaC construct, ask whether it's expressing a *capability requirement* or an *implementation choice* — only the former belongs here.

---

## tasks.md — Decisions and validation outcomes

Tasks describe what must be *decided*, *designed*, *deployed*, and *validated* — not implementation steps.

**Good tasks:**
- "Select the secondary Azure region — confirm availability zone support and data residency compliance"
- "Confirm the cross-region routing model from the design options and document rationale"
- "Validate that primary-region connectivity is unaffected after secondary deployment"

**Bad tasks (too implementation-prescriptive):**
- "Add `deploySecondaryRegion` bool param to `module.bicep`"
- "Run `az network nic show-effective-route-table` on a test NIC"
- "Update `vars-global.yml` with secondary region variable"

Group tasks by phase matching the design's Migration Plan. Each task should be verifiable — you know when it's done without reading code.

---

## Guardrails

- Create ALL artifacts needed for implementation (as defined by schema's `apply.requires`)
- Always read dependency artifacts before creating a new one
- If context is critically unclear, ask the user — but prefer making reasonable decisions to keep momentum
- If a change with that name already exists, ask if the user wants to continue it or create a new one
- Never include implementation details (file paths, parameter names, CLI commands) in any artifact
- **IMPORTANT**: `context` and `rules` from `openspec instructions` are constraints for YOU — do NOT copy them into the output files
