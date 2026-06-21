---
name: semver-advisor
description: Analyze uncommitted and unmerged changes to recommend a semantic version bump (major/minor/patch/none), candidate release version, and a conventional commit message with rationale, risks, and confidence level. Advisory only — never creates commits or tags. Use when user asks for a version bump recommendation, commit message suggestion, pre-PR change review, or to decide whether changes warrant a release.
---

# Semantic Version and Commit Message Advisor

Advisory only. Never create commits, modify the git index, create or push tags.

## Workflow

1. **Get baseline version**: `git describe --tags --abbrev=0 --match "v[0-9]*.[0-9]*.[0-9]*" 2>/dev/null || echo "v0.0.0"`
2. **Collect all unmerged changes** (avoid double-counting):
   - `git diff HEAD --stat` — staged + unstaged working-tree changes
   - `git log origin/main..HEAD --oneline` — branch commits not yet in main
   - `git diff origin/main...HEAD --stat` — combined diff from merge base
3. **Group** changed files by functional area.
4. **Classify** each group — see [CLASSIFICATION.md](CLASSIFICATION.md).
5. **Select** the highest-impact classification: `major > minor > patch > none`.
6. **Calculate** candidate version (see table below).
7. **Generate** one conventional commit message for the dominant change.
8. **Flag** mixed-purpose changes that should be split into separate commits.

## Candidate Version Calculation

| Latest  | major  | minor  | patch  | none       |
|---------|--------|--------|--------|------------|
| v1.4.2  | v2.0.0 | v1.5.0 | v1.4.3 | no release |
| v0.0.0  | v1.0.0 | v0.1.0 | v0.0.1 | no release |

## Commit Message Format

```
<type>(<scope>): <imperative summary> [<bump-type>]
```

| Classification | Preferred type |
|---|---|
| major | `feat`, `refactor`, `break` |
| minor | `feat` |
| patch | `fix` |
| none | `docs`, `test`, `chore`, `ci`, `style`, `refactor` |

Infer scope from the changed area: `networking`, `configuration`, `pipelines`, `governance`, `tests`, etc.

## Required Output

```
## Version Recommendation
Latest stable version: vX.Y.Z
Candidate version: vX.Y.Z or no release
Recommended bump: major | minor | patch | none
Confidence: high | medium | low

## Suggested Commit Message
<type>(<scope>): <summary> [<bump-type>]

## Change Assessment
- <area>: <classification> — <reason>

## Decision Rationale
<Why the selected classification is the highest applicable impact.>

## Risks or Review Needed
- <Potential breaking change, mixed concern, missing context, or suggested split.>
```

## Hard Constraints

- Do not create commits, tags, or modify any file.
- Do not assume a candidate version is an official release.
- Do not silently classify a potential breaking change as `patch` or `none`.
- Do not hide mixed changes that warrant separate commits.

See [CLASSIFICATION.md](CLASSIFICATION.md) for classification rules, confidence guidance, and worked examples.
