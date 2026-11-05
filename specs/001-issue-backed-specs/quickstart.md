# Quickstart: validating issue-backed specs

Prerequisites: Backstage with `@backstage/plugin-scaffolder-backend-module-github` (provides `github:issues:create`) and the speckit module; a sandbox Component whose repository you can create issues in.

1. **Dry run**: In the template editor, run `template-spec.yaml` with all fields filled. Expect `create-issue` to output `issueUrl` `.../issues/42` and the speckit step to receive that URL.
2. **Optional fields omitted**: Dry run with only the required fields. Expect valid rendering and no empty headings in the issue body.
3. **Real run (sandbox)**: Expect exactly one new open issue created by you, labelled `spec` and the chosen type; a draft spec PR whose body has `Refs #<n>`; and an output link to the issue.
4. **Failure**: Run with a user lacking issue permission. Expect failure at `create-issue`, no branch, no PR.
5. **Rerun**: Run again for the same feature. Expect a second, separate issue.
6. **No auto-close**: Merge the PR. Expect the issue to stay open.
