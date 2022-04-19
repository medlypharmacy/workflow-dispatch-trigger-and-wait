# GitHub Action for Dispatching Workflows

This action triggers another GitHub Actions workflow, using the `workflow_dispatch` event.  
The workflow must be configured for this event type e.g. `on: [workflow_dispatch]`

This allows you to chain workflows, the classic use case is have a CI build workflow, trigger a CD release/deploy workflow when it completes. Allowing you to maintain separate workflows for CI and CD, and pass data between them as required.

For details of the `workflow_dispatch` even see [this blog post introducing this type of trigger](https://github.blog/changelog/2020-07-06-github-actions-manual-triggers-with-workflow_dispatch/)

_Note 1._ The GitHub UI will report flows triggered by this action as "manually triggered" even though they have been run programmatically via another workflow and the API

_Note 2._ If you want to reference the target workflow by ID, you will need to list them with the following REST API call `curl https://api.github.com/repos/{{owner}}/{{repo}}/actions/workflows -H "Authorization: token {{pat-token}}"`

_This action is a fork of `aurelien-baudet/workflow-dispatch` to add support for customized job name of workflow triggered by workflow dispatch._

## Inputs

### `workflow`

**Required.** The name or the filename or ID of the workflow to trigger and run.



All values must be strings (even if they are used as booleans or numbers in the triggered workflow). The triggered workflow should use `fromJson` function to get the right type

### `repo`

**Optional.** The default behavior is to trigger workflows in the same repo as the triggering workflow, if you wish to trigger in another GitHub repo "externally", then provide the owner + repo name with slash between them e.g. `microsoft/vscode`

### `ref`

**Required.** The Git reference used with the triggered workflow run. The reference can be a branch, tag, or a commit SHA. If omitted the context ref of the triggering workflow is used. If you want to trigger on pull requests and run the target workflow in the context of the pull request branch, set the ref to `${{ github.event.pull_request.head.ref }}`. **Note please give default branch name `master` or `main`. Default value is `master`.**

### `token`

**Required.** A GitHub access token (PAT) with write access to the repo in question. **NOTE.** The automatically provided token e.g. `${{ secrets.GITHUB_TOKEN }}` can not be used, GitHub prevents this token from being able to fire the `workflow_dispatch` and `repository_dispatch` event. [The reasons are explained in the docs](https://docs.github.com/en/actions/reference/events-that-trigger-workflows#triggering-new-workflows-using-a-personal-access-token).

The solution is to manually create a PAT and store it as a secret e.g. `${{ secrets.PERSONAL_TOKEN }}`

### `repo-name`:

**Required.** Repository Name which invokes another workflow. Always set its value to be `${{github.repository}}`.

### `actor-name`:

**Required.** Name of the user who want to trigger workflow. Always set its value to be `${{github.actor}}`.

### `ref-name`:

**Required.** The pull request that triggered this workflow. Always set its value to be `${{github.ref}}`.

### `head-ref-name`:

**Required.** The name of the head branch. Always set its value to be `${{github.head_ref}}`.

### `base-ref-name`:

**Required.** The name of the base branch. Always set its value to be `${{github.base_ref}}`.

### `inputs`

**Optional.** The inputs to pass to the workflow (if any are configured), this must be a JSON encoded string, e.g. `{ "myInput": "foobar" }`.

### `wait-for-completion`

**Optional.** If `true`, this action will actively poll the workflow run to get the result of the triggered workflow. It is `true` or enabled by default. If the triggered workflow fails due to either `failure`, `timed_out` or `cancelled` then the step that has triggered the other workflow will be marked as failed too.

### `wait-for-completion-timeout`

**Optional.** The time to wait to mark triggered workflow has timed out. The time must be suffixed by the time unit e.g. `10m`. Time unit can be `s` for seconds, `m` for minutes and `h` for hours. It has no effect if `wait-for-completion` is `false`. Default is `1h`

### `wait-for-completion-interval`
            base-ref-name: ${{github.base_ref}}
            # Optional Inputs
            inputs: '{ "message": "hello!" }'
            wait-for-completion: true
            wait-for-completion-interval: 10s
            wait-for-completion-timeout: 30s
```

Workflow `e2e.yml` in app-e2e repo

```yaml
name: E2E Tests

on:
  workflow_dispatch:
    inputs:
      repo-branch:
        description: "Branch of repository which invokes this workflow"
        required: true
      workflow-dispatch-details:
        description: "Workflow Dispatch Details"
        required: true
      repo-name:
        description: "Repository Name"
        required: true
      ref-name:
        description: "The pull request that triggered this workflow."
        required: true

jobs:
  build:
    name: Repo:${{github.event.inputs.repo-name}}   Ref:${{github.event.inputs.ref-name }}
    runs-on: ubuntu-latest
    steps:
      - name: Workflow Dispatch Details
        run: |
          echo "${{ github.event.inputs.workflow-dispatch-details }}"
```

## Want to contribute

Run the following three commands before raising the PR.

```
npm install -g yarn
yarn install
yarn run build
```
