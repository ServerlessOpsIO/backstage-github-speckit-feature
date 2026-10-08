# Spec Kit Feature Backstage Template

This repository is a Backstage scaffolder template that uses [Spec Kit](https://github.com/github/spec-kit) and the GitHub Copilot cloud agent to take a new feature for an existing software component from a description to a set of tasks, and optionally to an implementation.

Unlike the project templates, it does not create a new repository. It runs a series of Copilot agent tasks against a component's existing GitHub repository.

## What this template does

The template runs these Spec Kit commands in order. Each one runs as a Copilot agent task, and the template waits for each task to complete before starting the next:

1. `specify`: writes a feature specification from the description you enter
2. `plan`: creates a technical implementation plan for the feature
3. `checklist`: generates a quality checklist for the specification
4. `tasks`: breaks the plan into tasks
5. `taskstoissues`: turns the tasks into GitHub issues
6. `analyze`: checks the specification, plan, and tasks for consistency and coverage

If you choose to implement the feature, two more commands run:

7. `implement`: implements the tasks
8. `converge`: brings the feature artifacts together; the template does not wait for this task to finish

Each task starts from the branch that the previous task worked on, so every step builds on the output of the step before it.

## Prerequisites

- The Backstage instance has the `scaffolder-backend-module-github-copilot` module installed, which provides the `github:copilot:speckit:*` actions.
- The component's repository already exists on GitHub and has Spec Kit initialized for Copilot.
- The user running the template has a GitHub Copilot Business or Copilot Enterprise subscription. The template asks the user to sign in to GitHub so it can launch Copilot tasks on their behalf.

## Using the template

The template asks for:

- the existing component to add the feature to
- a description of the feature to build, which is sent to `specify`
- optional technical guidance, such as technology choices or constraints, which is sent to `plan`
- whether Copilot should implement the feature after the tasks are created

The template finds the component's repository by its name in the `ServerlessOpsIO` GitHub organization, so the repository name must match the component name.

The template can take a long time to run, because each step waits for a Copilot task to finish. If any task ends in a state other than `completed`, the template stops at that step.

## Repository layout

- `template.yaml`: the Backstage scaffolder template definition
