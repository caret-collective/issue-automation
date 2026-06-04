<!-- Project Header -->
<div align="center">
	<h1 class="projectName">Issue Automation</h1>
	<p class="projectBadges info">
		<img src="https://johng.io/badges/category/Other.svg" alt="Project category" title="Project category">
		<img src="https://img.shields.io/github/languages/top/caret-collective/issue-automation.svg" alt="Language" title="Language">
		<img src="https://img.shields.io/github/repo-size/caret-collective/issue-automation.svg" alt="Repository size" title="Repository size">
		<a href="LICENSE"><img src="https://img.shields.io/github/license/caret-collective/issue-automation.svg" alt="Project license" title="Project license"/></a>
		<a href="https://conventionalcommits.org"><img src="https://img.shields.io/badge/Conventional%20Commits-1.0.0-%23FE5196?logo=conventionalcommits&logoColor=white" alt="Conventional Commits" title="Conventional Commits"/></a>
	</p>
	<p class="projectDesc">
		Reusable GitHub Actions workflows and composite actions for automating issue lifecycle events across a GitHub organization
	</p>
	<br/>
</div>

## 👋 About

A set of reusable GitHub Actions workflows and composite actions that keep org-level issue fields and project membership in sync as issues move through their lifecycle. GitHub's built-in project automations cannot update org-level issue fields or add issues to projects across all repos in an org at once, so this repo fills that gap.

> [!NOTE]
> Syncing the `Stage` field to `Blocked` when an issue dependency is created or removed is not currently supported because the `issue_dependencies` webhook event is not a valid GitHub Actions trigger. This has to be done manually for now.

### Features

- ♾️ Automations covering the full issue lifecycle
- ⚡ Zero-config setup via org-level variables
- 🔐 `GITHUB_TOKEN` is used for all automations (except adding issues to projects, which requires a PAT with project write scope)
- 🧩 Composite actions can be used independently for custom workflows

### Assumptions

If you want to use these automations with your own organization, you'll have to make sure your workflow and org setup are similar to ours. If not, you can always use the individual composite actions directly in your own workflow and adapt the inputs to match your setup.

#### Issue fields

This project assumes your org uses issue fields to track work metadata rather than project fields. That lets the same issue move across multiple projects while keeping status and triage data attached to the issue itself.

> [!TIP]
> `Stage` is used because `Status` and `State` are both reserved field names in GitHub.

A single-select `Stage` field is used to represent work status, with values for ready, in progress, and blocked. The exact labels do not need to match ours because they are supplied through org variables. Issues that are waiting for triage or already closed do not need a `Stage` value, since the open or closed state is used instead.

Separate `Priority` and `Effort` issue fields are also expected, with whatever option values your org already uses.

#### Projects

The automation assumes new issues should be added to an org project, not just updated within a single repository project.

#### Compatibility

The reusable workflow depends on org-level variables, so your organization needs to support them in the repositories where the workflow runs. Free organizations may be limited to public repositories, while paid organizations can use the same setup in private repositories.

### Automations

| Trigger                          | Action                           |
| -------------------------------- | -------------------------------- |
| Issue assigned                   | Sets `Stage` to `🚧 in progress` |
| Issue unassigned (last assignee) | Sets `Stage` to `🟢 ready`       |
| Issue closed                     | Clears `Stage`                   |
| Issue opened                     | Adds issue to the `Global` project |
| Effort and Priority fields set   | Sets `Stage` to `🟢 ready`       |

## 📦 Installation

### 1. Set org-level variables

Go to your org settings and add the following variables. Every repo that uses the caller workflow will automatically pick these up.

| Variable                  | Description                                               |
| ------------------------- | --------------------------------------------------------- |
| `STAGE_FIELD_ID`          | Numeric ID of the `Stage` issue field                     |
| `STAGE_READY_VALUE`       | Value used for the ready state in the `Stage` field       |
| `STAGE_IN_PROGRESS_VALUE` | Value used for the in progress state in the `Stage` field |
| `EFFORT_FIELD_ID`         | Numeric ID of the `Effort` issue field                    |
| `PRIORITY_FIELD_ID`       | Numeric ID of the `Priority` issue field                  |
| `PROJECT_NUMBER`          | Number of the project to add new issues to                |

To find a field's numeric ID, look at the URL when editing the issue field or run:

```sh
gh api \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  /orgs/YOUR_ORG/issue-fields \
  --jq '.[] | {id, name}'
```

### 2. Set the `ORG_PROJECT_TOKEN` secret

> [!IMPORTANT]
> Fine-grained personal access tokens [can't be used to access GitHub Projects](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#fine-grained-personal-access-tokens-limitations) at the moment, so you'll have to use a classic PAT.

Adding issues to a project requires a token with project write scope, which `GITHUB_TOKEN` does not have. Create a **classic PAT** with `project` and `read:org` scopes and add it as an org-level secret named `ORG_PROJECT_TOKEN`.

> [!TIP]
> If you do not need the "add to project" automation, you can skip this step. The action will skip gracefully if `project_number` is not set.

### 3. Add the caller workflow to each repo

Copy `.github/workflows/issue-automation.yml` from this repo into `.github/workflows/` of each repo that should participate.

```yaml
name: 🤖 Issue Automation
run-name: '🤖 [issue ${{ github.event.action }}] ${{ github.event.issue.title }}'
on:
  issues:
    types:
      - opened
      - assigned
      - unassigned
      - closed
      - reopened
      - field_added
permissions:
  issues: write
jobs:
  automate:
    uses: caret-collective/issue-automation/.github/workflows/issue-automations.yml@main
    secrets:
      project_token: ${{ secrets.ORG_PROJECT_TOKEN }}
```

### 4. Profit!

Now, when an issue is opened, assigned, or closed, the automation should run.

## 🕹️ Usage

### Using actions directly

> [!IMPORTANT]
> The composite actions don't read org vars. You must pass the values explicitly.

Each automation is a standalone composite action you can call directly without the reusable workflow. This is useful if you only want a subset of automations or need to integrate with a more complex workflow. Refer to each action for its specific inputs.

```yaml
- uses: caret-collective/issue-automation/on-issue-closed@main
  with:
    stage_field_id: ${{ vars.STAGE_FIELD_ID }}
```

All actions except `on-issue-opened` use `GITHUB_TOKEN` automatically. For `on-issue-opened`, set `GH_TOKEN` explicitly to a classic PAT with `project` scope:

```yaml
- uses: caret-collective/issue-automation/on-issue-opened@main
  with:
    project_number: ${{ vars.PROJECT_NUMBER }}
  env:
    GH_TOKEN: ${{ secrets.ORG_PROJECT_TOKEN }}
```

Available actions:

| Action                | Inputs                                                                        |
| --------------------- | ----------------------------------------------------------------------------- |
| `on-issue-assigned`   | `stage_field_id`, `stage_in_progress_value`                                   |
| `on-issue-closed`     | `stage_field_id`                                                              |
| `on-issue-unassigned` | `stage_field_id`, `stage_ready_value`                                         |
| `on-issue-opened`     | `project_number`                                                              |
| `on-triage-complete`  | `effort_field_id`, `priority_field_id`, `stage_field_id`, `stage_ready_value` |

## 🤖 Advanced Usage

### Updating the API version

The GitHub REST API version is defined once in `issue-automations.yml` and inherited by all actions when called through the reusable workflow:

```yaml
env:
  GH_API_VERSION: '2026-03-10'
```

When GitHub releases a new stable version, update this value in `issue-automations.yml`. When actions are called directly without the reusable workflow, they fall back to their own hardcoded default.

See [GitHub's API version documentation](https://docs.github.com/en/rest/about-the-rest-api/api-versions) for the latest available version.

### Pinning to a release

The caller workflow and all action references use `@main`. You might want to pin a release tag for stability/security:

```yaml
uses: caret-collective/issue-automation/.github/workflows/issue-automations.yml@v1
```

## 🛟 Support

Need help? See the [support resources](https://github.com/caret-collective/.github/blob/main/docs/SUPPORT.md) for information on how to:

- request features
- report bugs
- ask questions
- report security vulnerabilities

### Troubleshooting

#### `on-issue-opened` fails with `unknown owner type` or a permissions error

`gh project item-add` uses the GitHub GraphQL API to resolve the project owner. This fails if the token lacks the visibility to look up the org. To fix this:

1. Confirm `ORG_PROJECT_TOKEN` is set as an org-level secret and has not expired
2. Ensure the token is a **classic PAT** (fine-grained tokens cannot access GitHub Projects)
3. Confirm the token has the `project` and `read:org` scopes
4. Confirm `PROJECT_NUMBER` is the correct project number (visible in the project URL)

`GITHUB_TOKEN` will never work for this action, even with `permissions: write-all`.

#### Enabling debug logging

All actions include a debug step that prints inputs and context when runner debug logging is enabled. To enable it, add a secret named `ACTIONS_STEP_DEBUG` with the value `true` to the repo or org, then re-run the workflow.

## 🤝 Contributing

Want to help out? Pull requests are welcome for:

- feature implementations
- bug fixes
- documentation
- tests

See the [contribution guide](../../contribute) for more details.

## 🧾 License

Copyright © 2026 [John Goodliff](https://johng.io/r/issue-automation) ([@twocaretcat](https://github.com/twocaretcat)).

This project is licensed under the MIT license. See the [license](LICENSE) for more details.

## 💕 Funding

Find this project useful? [Sponsoring me](https://johng.io/funding) will help me cover costs and **_commit_** more time to open-source.

If you can't donate but still want to contribute, don't worry. There are many other ways to help out, like:

- 📢 reporting (submitting feature requests & bug reports)
- 👨‍💻 coding (implementing features & fixing bugs)
- 📝 writing (documenting & translating)
- 💬 spreading the word
- ⭐ starring the project

I appreciate the support!
