# Contract: `github:issues:create` step

Provided by `@backstage/plugin-scaffolder-backend-module-github`.

```yaml
- id: create-issue
  name: Create spec tracking issue
  action: github:issues:create
  input:
    repoUrl: github.com?repo=${{ parameters.component | parseEntityRef | pick('name') }}&owner=${{ parameters.github_org }}
    title: ${{ parameters.spec_title }}
    body: <rendered from the spec form fields>
    labels:
      - spec
      - ${{ parameters.spec_type }}
    token: ${{ secrets.USER_GITHUB_TOKEN }}
  # outputs: issueUrl (string), issueNumber (number)
```

Downstream use:
- `github:speckit:workflow` `inputs.spec: ${{ steps['create-issue'].output.issueUrl }}` (runs `/speckit-specify <URL>`)
- Spec PR body: `Refs #${{ steps['create-issue'].output.issueNumber }}`
- Output link: "Spec tracking issue" -> `issueUrl`

Guarantees: the step runs first; failure stops the workflow before any artifacts are produced; the issue is never closed by the workflow.
