# Implementation Plan: Issue-Backed Specs

**Branch**: `001-issue-backed-specs` | **Date**: 2026-11-05 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-issue-backed-specs/spec.md`

## Summary

Change `template-spec.yaml` (`1-speckit-feature`) so every new spec is tracked by a GitHub issue. The Feature form becomes a single page of structured fields (FR-003/FR-004). A new first step uses the Backstage `github:issues:create` action, with the requester's token, to create an open issue in the component repository. The action's `issueUrl` output is passed to the Spec Kit specify command (`/speckit-specify <URL>`), so the spec is generated from the issue. If issue creation fails, the workflow stops before anything is cloned or generated, so no spec artifacts remain (FR-010). The issue number and link are added to the PR description and the template output links. The plan and work templates reference the issue in their PRs and never close it.

## Technical Context

**Language/Version**: Backstage scaffolder template YAML (`scaffolder.backstage.io/v1beta3`), Nunjucks expressions, inline POSIX shell (`set -eu`)

**Primary Dependencies**: `github:issues:create` from `@backstage/plugin-scaffolder-backend-module-github` (new); existing `x_github:*` actions, `github:speckit:workflow` (`scaffolder-backend-module-github-speckit`), `catalog:fetch`, `fetch:template`

**Storage**: Git files in the component repository plus a GitHub issue; no database

**Testing**: Backstage template editor dry run (`github:issues:create` supports dry run and returns `issues/42`); render checks of every form combination; sandbox Component run

**Target Platform**: Backstage scaffolder backend

**Project Type**: Backstage workflow templates (YAML only)

**Performance Goals**: N/A. The issue step adds one API call.

**Constraints**: Token only via `SoRequestUserCredentials` / `USER_GITHUB_TOKEN` secret; the issue must be created as the requester (FR-011); never auto-close; no push to the default branch

**Scale/Scope**: One template changed (`template-spec.yaml`), reference changes in `template-plan.yaml` and `template-work.yaml`, plus `README.md`

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-checked after Phase 1 design.*

| Principle | Status | Notes |
|---|---|---|
| I. Every input produces a valid change | PASS | Optional fields use `default` filters so rendering does not break. Every variable used by the skeleton is still supplied. Verify all optional-field combinations by dry run. |
| II. Catalog entities drive inputs | PASS | Catalog pickers are unchanged and stay in the first page. |
| III. Stable identities | PASS | The issue is created in the existing component repository, found by Component name and `github_org`. Branch naming is unchanged. |
| IV. Ordered, fail-safe steps | PASS | The issue step comes first, so a failure prevents all later steps. Optional fields are not required. |
| V. Secrets | PASS | The issue is created with the requester's token from `secrets.USER_GITHUB_TOKEN`. The token is never written to files or PR text. The `read:user` scope is kept. Issue creation needs repository access, which the existing GitHub credential scope request must cover (see research R3). |
| VI. Accurate forms/outputs | PASS | Output links gain a "Spec tracking issue" link. Field titles and descriptions follow the spec. |
| Quality gates | PASS | README update is in scope (new action and its providing module). |

Post-design re-check: no violations, so no Complexity Tracking entries are needed.

## Project Structure

### Documentation (this feature)

```text
specs/001-issue-backed-specs/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── spec-form.md
│   └── issue-action.md
└── tasks.md             # created by /speckit-tasks
```

### Source Code (repository root)

```text
template-spec.yaml        # new one-page form, github:issues:create step, specify <URL>, issue links
template-plan.yaml        # reference the spec issue in the PR (when a valid issue exists)
template-work.yaml        # reference the spec issue in the PR; never use closing keywords
skeleton/spec-catalog/catalog-info.yaml  # optional: record the issue URL as an annotation or link
README.md                 # document github:issues:create and its module; new form fields
```

**Structure Decision**: YAML-only change to the existing templates. The issue is created in `template-spec.yaml`. The plan and work templates only consume the issue reference, which they read from the spec Resource or from the spec directory.

## Complexity Tracking

No constitution violations.
