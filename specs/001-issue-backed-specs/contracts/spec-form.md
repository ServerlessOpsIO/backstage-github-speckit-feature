# Contract: one-page spec form

Page 1 (unchanged): hidden `github_org`, `owner`, `domain`, `system`, `component`.

Page 2 (all fields together):

| Property | Widget | Rows | Required |
|---|---|---|---|
| spec_type | radio (feature, bug, documentation) | - | yes |
| spec_title | text | - | yes |
| story | text, prompt "As a ____, I can ____" | - | yes |
| proposed_solution | textarea | 10 | yes |
| scope_boundaries | textarea | 5 | no |
| dependencies | textarea | 5 | no |
| additional_context | textarea | 5 | no |
| acceptance_criteria | textarea, default `- [ ] ...` | 5 | yes |
| github_auth | `SoRequestUserCredentials` -> `USER_GITHUB_TOKEN` | - | yes |

Replaces `speckit_specify_description` and `speckit_specify_prompt`.
