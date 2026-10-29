<!--
Sync Impact Report
Version change: unratified scaffold → 1.0.0
Modified principles: generic placeholder principles I–V → principles I–VI below.
Added sections: Workflow Template Constraints; Development Workflow and Quality Gates.
Removed sections: none.
Scope exclusions: Project-creation rules for rendering application skeletons and pipelines,
publishing repositories, catalog registration, repository defaults/properties, and optional
post-registration provisioning (such as Datadog) are omitted because this repository contains
only workflows for existing components. Project repository/entity naming rules are omitted
because these workflows target an existing component repository; the one generated entity is
a Resource with its own explicit naming rule below. GitHub Actions raw-block rules apply only
if a future skeleton contains Actions expressions.
Known alignment gaps (not changed in this constitution-only update): template-spec.yaml creates
a branch from each generated spec directory rather than reusing a prior branch on rerun;
skeleton/spec-catalog/catalog-info.yaml conditionally omits system and does not set lifecycle;
the first parameter page has a template-specific owner field in template-spec.yaml.
Deferred placeholders: none.
-->
# backstage-github-speckit-feature Constitution

## Core Principles

### I. Every workflow input produces a valid change
Every combination of template inputs MUST produce a valid, complete workflow result without
hand edits. Every variable used by a rendered file, including variables in file or directory
names, MUST be supplied to that `fetch:template` action. Conditional content MUST NOT be
referenced outside the condition that defines it. Generated YAML and JSON MUST remain valid for
all supported input combinations, including conditional commas and indentation. If a future
skeleton contains GitHub Actions expressions, expressions intended for Actions MUST be wrapped
in `{% raw %}...{% endraw %}`. A change to an input or skeleton MUST be verified by rendering
every affected option combination.

The only current rendered skeleton is `skeleton/spec-catalog/catalog-info.yaml`, used by
`template-spec.yaml` to create a catalog Resource. The plan and work templates do not render
skeletons or pipelines. Their branch changes, catalog metadata, and pull requests MUST be
valid and usable without manual repair.

### II. Catalog entities drive entity inputs
Catalog entity values MUST be selected through pickers with `allowArbitraryValues: false`,
rather than free-text fields. The shared context is the hidden `github_org`, followed by
Domain, System filtered by the selected Domain, and Component filtered by the selected System.
`template-spec.yaml` also selects the component owner with `MyGroupsPicker`. The plan and work
templates select a `Resource` of type `doc-spec` whose `dependsOn` relation points to the
selected Component. These shared catalog fields MUST retain the same order, picker behavior,
and relation filters across templates; template-specific fields MUST be placed separately.

Values read from `catalog:fetch` MUST come from the entity they describe. Any change to
`entityRefs` ordering or contents MUST update every result read by index. Template comments
MUST document implicit catalog contracts next to their use, including the repository-name
match to the Component name and the `serverlessops.io/spec-dir` annotation used to locate a
feature directory.

### III. Existing repositories and generated entities have stable identities
These templates MUST operate on an existing Component repository; they MUST NOT create a new
project repository. `template-spec.yaml`, `template-plan.yaml`, and `template-work.yaml`
derive the repository name from the selected Component's catalog name. The hidden `github_org`
parameter MUST default to `ServerlessOpsIO`; steps and skeletons MUST use that parameter rather
than hard-code the organization.

`template-spec.yaml` renders `skeleton/spec-catalog/catalog-info.yaml` as a `Resource` of type
`doc-spec`. Its entity name MUST follow
`<component-name>-spec-<spec-id>`, and its `github.com/project-slug` annotation MUST identify
the selected component repository. Every generated `catalog-info.yaml` MUST set `owner`,
`system`, and `lifecycle`, declare its `spec.type`, and express its catalog relationship to the
Component with `dependsOn` or `providesApis`. The generated Resource MUST retain its spec
directory annotation and repository links.

Workflow templates MUST NOT push to or merge into the repository's default branch. They MUST
push changes to named `spec/`, `plan/`, or `feature/` branches and open draft pull requests for
review. Re-running a workflow for the same feature MUST continue from its existing branch and
must not create duplicate work. Branch names MUST derive from the feature directory name.

### IV. Workflow steps are ordered and fail safely
Each workflow MUST fetch the catalog entities it needs and clone the selected component
repository before running work that depends on them. It MUST then create or check out its
named branch, run the applicable Spec Kit workflow, commit changes, push the branch, and open a
pull request. The spec workflow additionally renders and commits the spec Resource catalog
file; the plan workflow creates plan, checklist, and task artifacts; the work workflow creates
task issues and implements the selected feature. Follow-up analysis or clarification comments
MUST target the pull request created by that run.

Optional inputs MUST have honest defaults and MUST only be required when enabled. Conditional
fields MUST be controlled by schema dependencies and shown only for the applicable choice. In
`template-work.yaml`, implementation instructions MUST remain hidden unless
`add_implement_prompt` is true. Optional technical guidance in `template-plan.yaml` MUST remain
optional.

Inline shell in a workflow MUST start with `set -eu` and stop on unexpected state. A branch
lookup may distinguish an absent remote branch from an operational error, but MUST fail for
other unexpected statuses. Comments MUST explain non-obvious behavior, especially branch
reuse, catalog filters, and values passed explicitly to external actions.

### V. Credentials and secrets stay out of templates and changes
User credentials MUST be requested with `SoRequestUserCredentials` into
`USER_GITHUB_TOKEN`, with only the required scopes, and passed to actions through scaffolder
secrets. Tokens MUST NOT be written to files, git remotes, logs, commit content, or pull request
text. Inline shell that authenticates a remote operation MUST use the workflow-provided token
without persisting it.

Commits MUST use the profile display name and email of the user running the template. The
templates and rendered catalog files MUST contain no secret values or account-specific
credentials.

### VI. Forms and templates are accurate and consistent
Parameter titles, descriptions, defaults, and output labels MUST accurately describe their
behavior. Hidden fields MUST also have accurate titles and descriptions. An enum choice MUST
NOT be offered unless it produces a working result. Step IDs and names MUST describe their
actions.

The templates MUST share the catalog-context fields and picker conventions defined in
Principle II, metadata owner and tag conventions, and GitHub organization handling. The
existing templates are named `template-spec.yaml` (`1-speckit-feature`), `template-plan.yaml`
(`2-speckit-plan`), and `template-work.yaml` (`3-speckit-work`); their metadata tags are
`spec` and `workflow`, their catalog owner is `group:00gjdgxs1ur6lsu`, and their template
type is `component`. Any divergence in shared form fields or metadata MUST be explained where
it is defined.

Each template's `output.links` MUST link to the pull request and catalog entities involved,
and to other resources it creates. In particular, spec output links include the spec pull
request, spec branch content, the catalog Resource after merge, and the Component; plan output
links include its pull request, plan, spec Resource, and Component; work output links include
its pull request, task issues, spec Resource, and Component.

## Workflow Template Constraints
This repository contains three workflow templates for work on existing Components. The
specification workflow creates a feature spec and a `doc-spec` Resource catalog file; the plan
workflow creates plan, checklist, and task artifacts; the work workflow creates task issues and
implements the feature. None publishes a repository or registers a newly created project.

The only skeleton directory is `skeleton/spec-catalog/`, containing the catalog Resource
definition. There is no pipeline directory. Rules for generated application pipelines,
project-repository visibility and protection, repository custom properties, `main` defaults,
and `publish:github` or `catalog:register` settings do not apply to these workflow templates.
Any future project-creation template requires a constitution amendment defining its additional
contracts before it is introduced.

## Development Workflow and Quality Gates
Before merge, authors MUST:
- Run every affected template through the Backstage template editor dry run.
- Render every affected input combination and verify it against Principle I.
- Exercise changes to workflow git behavior or external actions against a test Component or
  sandbox.
- List the checks and runs performed in the pull request.

Changes to template inputs, generated files, required catalog entities, secrets, Backstage
actions, or other prerequisites MUST update `README.md` in the same change. A new or changed
scaffolder action MUST be identified in the README together with the Backstage module that
provides it. Generated, OS, and editor files, including `.DS_Store`, `__pycache__`, and
`node_modules`, MUST NOT be committed.

Work MUST use `feature/` or `bug/` branches that reference an issue number when one exists.
Commit messages MUST use conventional-commit prefixes such as `feat:`, `fix:`, or
`refactor:`. Changes MUST merge through pull requests.

## Governance
This constitution supersedes conflicting repository conventions. Every specification, plan,
and pull request MUST be checked for compliance with it. Amendments MUST be made in the same
change that introduces a new rule and MUST include a Sync Impact Report.

Versioning follows semantic versioning: MAJOR removes or incompatibly redefines a principle;
MINOR adds a principle or materially expands a section; PATCH clarifies wording or makes a
non-semantic refinement. Amendments MUST be reviewed and approved through a pull request.

**Version**: 1.0.0 | **Ratified**: 2026-10-29 | **Last Amended**: 2026-10-29
