# Classification Rules

## Impact Levels

| Classification | Meaning | Candidate version effect |
|---|---|---|
| `major` | Breaking consumer-facing change | Increment major, reset minor and patch to 0 |
| `minor` | New backward-compatible capability | Increment minor, reset patch to 0 |
| `patch` | Backward-compatible defect correction | Increment patch |
| `none` | No consumer-facing release impact | No version increment |

When multiple classifications exist, use the highest: `major > minor > patch > none`.

---

## Classification Guidance

### Major — breaking change

Use when the diff requires consumers to update their code, config, or automation.

- Removing or renaming a public API, parameter, output, command, or module
- Changing a configuration or IaC contract incompatibly
- Removing support for an existing pipeline-template input
- Altering default behavior in a way that breaks existing usage

Commit marker: `[major]`

### Minor — new backward-compatible capability

Use when the diff adds new functionality without breaking anything existing.

- Adding an optional parameter, feature, command, resource type, report, or capability
- Adding a new integration that does not affect existing consumers
- Extending a schema with optional fields
- Adding a new reusable pipeline or IaC capability

Commit marker: `[minor]`

### Patch — backward-compatible fix

Use when the diff corrects released behavior without changing the supported contract.

- Fixing deployment, validation, authentication, or runtime behavior
- Correcting an incorrect default or calculation
- Fixing a regression
- Correcting user-facing documentation required to use the released functionality correctly

Commit marker: `[patch]`

### None — no consumer-facing impact

Use when changes do not affect released consumer behavior.

- Internal refactoring with no behavioral change
- Formatting, linting, or spelling changes
- Internal test-only updates
- CI maintenance with no released artifact impact
- Development-tooling changes
- Internal documentation changes

Commit marker: `[none]`

---

## Mixed Change Rule

When the change set contains unrelated work, identify whether it should be split and flag it.

Example:
```
- Adds a new feature      → minor
- Fixes an unrelated bug  → patch
- Updates CI formatting   → none
```

Output: recommend `minor` (highest wins), flag that patch and CI changes should be separate commits where practical.

---

## Confidence Levels

| Confidence | Meaning |
|---|---|
| High | The functional impact is clear from the diff. |
| Medium | The likely impact is clear, but consumer usage is partially unknown. |
| Low | The diff does not provide enough evidence to determine compatibility safely. |

### Escalation rules

- When a breaking change is possible but cannot be confirmed, do not silently classify as `patch` or `none`. State: _"Potential breaking change detected. Recommended classification: minor pending consumer-impact review. Confidence: low."_
- When all changes appear internal, use `none` rather than defaulting to `minor`.
- When changes appear consumer-facing but classification is unclear, recommend `minor` and flag the uncertainty.

---

## Worked Examples

### New Capability

```
## Version Recommendation
Latest stable version: v1.4.2
Candidate version: v1.5.0
Recommended bump: minor
Confidence: high

## Suggested Commit Message
feat(subscription-vending): add subscription ownership validation [minor]

## Change Assessment
- Subscription validation module: minor — adds a new backward-compatible validation capability.
- Unit tests: none — validates the new behavior only.

## Decision Rationale
The change introduces a new optional capability without removing or changing existing consumer contracts.

## Risks or Review Needed
- None identified.
```

### Internal Pipeline Maintenance

```
## Version Recommendation
Latest stable version: v1.4.2
Candidate version: no release
Recommended bump: none
Confidence: high

## Suggested Commit Message
ci(pipelines): update linting task configuration [none]

## Change Assessment
- CI configuration: none — internal pipeline maintenance only.
- Formatting: none — no runtime or consumer impact.

## Decision Rationale
No released behavior, public interface, artifact contract, or consumer workflow changes.

## Risks or Review Needed
- Confirm the pipeline change does not alter published artifact contents.
```

### Breaking Configuration Change

```
## Version Recommendation
Latest stable version: v1.4.2
Candidate version: v2.0.0
Recommended bump: major
Confidence: high

## Suggested Commit Message
feat(configuration): remove legacy environment schema [major]

## Change Assessment
- Configuration schema: major — removes fields used by existing consumers.
- Migration documentation: none — documents the required upgrade path only.

## Decision Rationale
Consumers using the removed schema must update their configuration before adopting this release.

## Risks or Review Needed
- Provide migration guidance.
- Confirm affected repositories and consumers before merging.
```
