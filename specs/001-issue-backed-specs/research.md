# Research: Issue-Backed Specs

## R1. Issue creation mechanism
- **Decision**: Use the Backstage `github:issues:create` action (`@backstage/plugin-scaffolder-backend-module-github`), as requested.
- **Rationale**: Inputs are `repoUrl`, `title`, `body`, `labels`, `assignees`, `milestone` and an optional `token`. Outputs are `issueUrl` and `issueNumber`. It supports dry run (returns `.../issues/42`) and checkpoints on `owner.repo.title`.
- **Alternatives**: `gh issue create` in shell (rejected: user asked for the action, and shell would handle tokens manually); Octokit custom action (rejected: unnecessary).

## R2. Passing the issue to specify
- **Decision**: Pass `${{ steps['create-issue'].output.issueUrl }}` as the `spec` input of the speckit workflow, so the step runs `speckit.specify` with the bare URL (`/speckit-specify <URL>`).
- **Rationale**: The issue body holds all authored fields, so the URL alone is sufficient and the issue stays the source of truth.
- **Alternatives**: Also pass the prompt text (rejected: duplicates content, can drift from the issue).

## R3. Requester identity (FR-011)
- **Decision**: Set `token: ${{ secrets.USER_GITHUB_TOKEN }}` on the action. The user's token comes from `SoRequestUserCredentials`.
- **Rationale**: Without a token the action falls back to the integration credentials, so the issue would be attributed to the Backstage app. The `github:issues:create` credential lookup requires repository write/issue access, so confirm that the existing GitHub credential request covers it (add `repo` or the minimum needed scope in `additionalScopes` if testing shows it is missing).
- **Alternatives**: Integration credentials (rejected: wrong creator).

## R4. Failure handling (FR-010)
- **Decision**: Make `create-issue` the first step, before the clone. Scaffolder stops on the first failed step.
- **Rationale**: The spec is generated on a throwaway workspace clone and is never pushed, so nothing is retained. The failure message should be actionable (permissions, repository name).
- **Alternatives**: Cleanup step on failure (rejected: nothing to clean up if the order is right).

## R5. One issue per submission (clarification)
- **Decision**: No lookup for existing issues. Each run creates a new issue.
- **Note**: The action's checkpoint key is `owner.repo.title`, which applies only to retries of the same task run, so it does not block new submissions with the same title.

## R6. Labels
- **Decision**: Labels `spec` and the selected type (`feature`, `bug`, `documentation`).
- **Note**: The action drops labels silently if the user lacks push access. Missing labels do not fail the run.

## R7. No auto-close (FR-002, FR-009)
- **Decision**: PR text uses `Refs #N` only, never `Closes`, `Fixes` or `Resolves`. No step calls an issue update or close API.

## R8. Form (FR-003, FR-004)
- **Decision**: One page, with the catalog pickers moved to the first page as before. The second page holds: `spec_type` (enum, `ui:widget: radio`), `spec_title` (string), `story` (string, placeholder/description "As a ____, I can ____"), `proposed_solution` (textarea, rows 10), `scope_boundaries`, `dependencies`, `additional_context` (textarea, rows 5, optional), `acceptance_criteria` (textarea, rows 5, default `- [ ] ...`, required), plus `github_auth`.
- **Note**: The default `- [ ] ...` alone would pass a required check. Add a `pattern` or `minLength` to require that the user changes it.

## R9. Existing short description and PR linkage
- **Decision**: Use the story sentence as the catalog Resource description. Add the issue reference to the spec PR body. Add the issue link to the template output.
- **Plan/work templates**: Locate the issue number from the spec PR or Resource, and add `Refs #N` to their PR text only when a valid number is found; otherwise report that linkage was not applied (FR-007).
