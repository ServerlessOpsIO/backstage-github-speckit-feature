# Tasks: Issue-Backed Specs

**Input**: Design documents from `/specs/001-issue-backed-specs/`

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/spec-form.md, contracts/issue-action.md, quickstart.md

**Tests**: No automated tests were requested. Validation is by Backstage template editor dry runs and render checks (see quickstart.md and the Polish phase).

**Organization**: Grouped by user story. This is a YAML-only change to `template-spec.yaml`, `template-plan.yaml`, `template-work.yaml`, `skeleton/spec-catalog/catalog-info.yaml` and `README.md`.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: User story (US1, US2, US3)

## Phase 1: Setup

**Purpose**: Confirm current state before editing.

- [X] T001 Review `template-spec.yaml`, `template-plan.yaml`, `template-work.yaml` and `skeleton/spec-catalog/catalog-info.yaml` against constitution principles I, IV, V and VI in `.specify/memory/constitution.md`, noting every use of `speckit_specify_description` and `speckit_specify_prompt` (all are removed by this feature)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: The one-page form that every story's inputs depend on.

**⚠️ CRITICAL**: The issue step (US1) consumes these fields.

- [X] T002 Replace the `Feature` page properties in `template-spec.yaml` (remove `speckit_specify_description` and `speckit_specify_prompt`) with: `spec_type` (enum `feature`, `bug`, `documentation`, `ui:widget: radio`, required), `spec_title` (single-line string, required), `story` (single-line string, required, description/placeholder "As a ____, I can ____"), `proposed_solution` (`ui:widget: textarea`, `ui:options.rows: 10`, required), `scope_boundaries`, `dependencies` and `additional_context` (each `ui:widget: textarea`, `rows: 5`, optional), `acceptance_criteria` (`ui:widget: textarea`, `rows: 5`, `default: "- [ ] ..."`, required). Add `spec_type`, `spec_title`, `story`, `proposed_solution`, `acceptance_criteria` and `github_auth` to the page `required` list
- [X] T003 In `template-spec.yaml`, add a `pattern` (or `minLength`) on `acceptance_criteria` so the untouched default `- [ ] ...` is rejected, with a clear validation message (research R8)
- [X] T004 In `template-spec.yaml`, keep the `github_auth` `SoRequestUserCredentials` field (`secretsKey: USER_GITHUB_TOKEN`) and confirm its `additionalScopes` grants issue creation access; add the minimum extra GitHub scope if needed (research R3, constitution principle V)

**Checkpoint**: The form renders on one page with all fields (FR-003, FR-004).

---

## Phase 3: User Story 1 - Create a tracked spec (Priority: P1) 🎯 MVP

**Goal**: Every new spec creates an open GitHub issue as the requester, and specify runs from that issue.

**Independent Test**: Dry run and sandbox run `template-spec.yaml`; one open issue is created by the requester, its link is in the outputs, and a failure at issue creation leaves no branch, PR or artifacts.

### Implementation for User Story 1

- [X] T005 [US1] Add a first step `create-issue` (action `github:issues:create`) in `template-spec.yaml`, placed before `clone`, per contracts/issue-action.md: `repoUrl: github.com?repo=${{ parameters.component | parseEntityRef | pick('name') }}&owner=${{ parameters.github_org }}`, `title: ${{ parameters.spec_title }}`, `labels: [spec, ${{ parameters.spec_type }}]`, `token: ${{ secrets.USER_GITHUB_TOKEN }}` (FR-001, FR-011, FR-010)
- [X] T006 [US1] In the `create-issue` step of `template-spec.yaml`, build `body` from the form fields with headings in this order: Story, Type, Proposed solution, Acceptance criteria, then Scope boundaries, Dependencies, Additional context, each of the last three only when provided (use `{% if %}` or `default` guards so no empty headings render; constitution principle I)
- [X] T007 [US1] In `template-spec.yaml`, change the `speckit-workflow` step input `spec` to `${{ steps['create-issue'].output.issueUrl }}` so specify runs `/speckit-specify <issue URL>` (research R2)
- [X] T008 [US1] In `template-spec.yaml`, update the `spec-catalog` step values: `description` from `parameters.story` (research R9), `spec_title` unchanged, and add `issue_url: ${{ steps['create-issue'].output.issueUrl }}` and `issue_number: ${{ steps['create-issue'].output.issueNumber }}`
- [X] T009 [P] [US1] In `skeleton/spec-catalog/catalog-info.yaml`, add annotation `serverlessops.io/spec-issue: ${{ values.issue_number | dump }}` and a link titled "Spec tracking issue" to `values.issue_url`, so downstream plan/work templates can read the issue (FR-005); keep the file valid YAML
- [X] T010 [US1] In `template-spec.yaml` `output.links`, add a "Spec tracking issue" link (icon `github`) to `${{ steps['create-issue'].output.issueUrl }}` so the issue number and link are visible in the outputs (acceptance scenario 2)
- [X] T011 [US1] Add a comment on the `create-issue` step in `template-spec.yaml` stating that it runs first so a failure stops the workflow before any clone, branch or PR exists (FR-010, constitution principle IV)

**Checkpoint**: User Story 1 works on its own.

---

## Phase 4: User Story 2 - Keep code review connected to tracked work (Priority: P2)

**Goal**: Spec, plan and work PRs reference the issue when a valid number exists.

**Independent Test**: Open each PR type for a spec with an issue and see `Refs #N`; for a spec without one, the PR still opens and the output reports linkage was not applied.

### Implementation for User Story 2

- [X] T012 [US2] In `template-spec.yaml`, rewrite the `open-spec-pr` `description` to start with `Refs #${{ steps['create-issue'].output.issueNumber }}` followed by the story and the spec fields (replacing the removed `speckit_specify_description` / `speckit_specify_prompt` references); never use `Closes`, `Fixes` or `Resolves` (FR-006, FR-002)
- [X] T013 [P] [US2] In `template-plan.yaml`, read `serverlessops.io/spec-issue` from `steps['fetch-spec'].output.entity.metadata.annotations` and add `Refs #<n>` to the `open-plan-pr` description only when the annotation holds a valid number; otherwise omit it without breaking rendering (FR-006, FR-007)
- [X] T014 [P] [US2] In `template-work.yaml`, make the same change to the implementation PR `description`, so every PR for the same spec submission keeps the reference (FR-008, SC-005)
- [X] T015 [US2] In `template-plan.yaml` and `template-work.yaml`, when no valid issue number exists, add a visible note to the PR description (or a template output text) saying issue linkage could not be applied (FR-007)
- [X] T016 [P] [US2] In `template-plan.yaml` and `template-work.yaml` `output.links`, add a "Spec tracking issue" link only when the annotation exists

**Checkpoint**: User Stories 1 and 2 both work.

---

## Phase 5: User Story 3 - Preserve human control over completion (Priority: P3)

**Goal**: No workflow step ever closes or edits the issue state.

**Independent Test**: Merge a linked PR and confirm the issue stays open.

### Implementation for User Story 3

- [ ] T017 [US3] Audit `template-spec.yaml`, `template-plan.yaml`, `template-work.yaml` and the PR comment bodies for `close`, `closes`, `fix`, `fixes`, `resolve`, `resolves` followed by an issue reference, and for any issue update or close call; remove any found (FR-002, FR-009)
- [ ] T018 [US3] Add a short comment next to each `Refs #` use in `template-spec.yaml`, `template-plan.yaml` and `template-work.yaml` stating that closing keywords must not be used because a human closes the issue

**Checkpoint**: All stories complete.

---

## Phase 6: Polish & Cross-Cutting Concerns

- [ ] T019 [P] Update `README.md`: document the `github:issues:create` action and its providing module `@backstage/plugin-scaffolder-backend-module-github`, the new one-page form fields, the issue-per-submission behaviour, and the no-auto-close rule (constitution quality gates)
- [ ] T020 Render every optional-field combination of the `create-issue` body and the `spec-catalog` skeleton (all optional fields present, each absent, all absent) and confirm valid output with no empty headings or broken YAML (constitution principle I)
- [ ] T021 Run quickstart.md scenarios 1–2 as dry runs in the Backstage template editor (issue URL `.../issues/42` passed to specify; optional fields omitted)
- [ ] T022 Run quickstart.md scenarios 3–6 in a sandbox Component (real issue by the requester with labels, failure without issue permission leaves no branch or PR, rerun makes a second issue, merge leaves the issue open)
- [ ] T023 Verify YAML validity of all changed files (for example `python3 -c "import yaml,sys; yaml.safe_load(open(sys.argv[1]))" template-spec.yaml` after substituting placeholders, or the template editor's validation)

---

## Dependencies & Execution Order

- Setup (T001) → Foundational (T002–T004) → US1 (T005–T011) → US2 (T012–T016) → US3 (T017–T018) → Polish (T019–T023)
- US2 depends on US1's skeleton annotation (T009) for T013/T014; T012 depends on T005
- US3 is an audit of the final text, so it runs after US1 and US2
- T002–T004, T005–T008, T010–T012 and T015 edit `template-spec.yaml` or share files, so they run sequentially

### Parallel Opportunities

- T009 (skeleton) in parallel with T005–T008 (`template-spec.yaml`)
- T013, T014 and T016 (different files) in parallel after T009
- T019 (README) in parallel with any phase after T002

## Implementation Strategy

### MVP First (User Story 1)

1. T001, then the form (T002–T004)
2. US1 (T005–T011)
3. Stop and validate with quickstart scenarios 1–4

### Incremental Delivery

1. Add US2 (PR references), validate with a plan and work run
2. Add US3 audit
3. Finish with README and full validation

## Notes

- Never use `Closes`, `Fixes` or `Resolves`; use `Refs #N` only
- The token comes only from `secrets.USER_GITHUB_TOKEN`; never write it to files or PR text
