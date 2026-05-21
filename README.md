<!-- Project Header -->
<div align="center">
	<h1 class="projectName">Issue Field Workflows</h1>
	<p class="projectBadges info">
		<img src="https://johng.io/badges/category/Other.svg" alt="Project category" title="Project category">
		<img src="https://img.shields.io/github/languages/top/caret-collective/issue-field-workflows.svg" alt="Language" title="Language">
		<img src="https://img.shields.io/github/repo-size/caret-collective/issue-field-workflows.svg" alt="Repository size" title="Repository size">
		<a href="LICENSE"><img src="https://img.shields.io/github/license/caret-collective/issue-field-workflows.svg" alt="Project license" title="Project license"/></a>
		<a href="https://conventionalcommits.org"><img src="https://img.shields.io/badge/Conventional%20Commits-1.0.0-%23FE5196?logo=conventionalcommits&logoColor=white" alt="Conventional Commits" title="Conventional Commits"/></a>
	</p>
	<p class="projectDesc">
		Reusable GitHub Actions workflows for keeping org-level issue fields in sync with issue lifecycle events.
	</p>
	<br/>
</div>

> [!NOTE]
> These workflows are specific to the [Caret Collective](https://github.com/caret-collective) organization and will require some customization if you want to use them in your own org.

## 👋 About

A collection of reusable GitHub Actions workflows for syncing GitHub's org-level issue fields when issues change state. GitHub org-level issue fields can't be updated by built-in automations or project workflows at this time. This repo provides a set of reusable workflows to fill that gap.

> [!NOTE]
> Syncing the `Stage` field to `Blocked` when an issue dependency is created or removed is not currently possible with GitHub Actions since the `issue_dependencies` webhook event is not a supported Actions trigger, so this need sto be done manually.

Currently, only one workflow exists: `clear-stage-field-on-issue-close.yml`. This workflow clears the `Stage` field automatically when an issue is closed (we use `Stage` because `Status` and `State` are reserved names).

### Features

- uses `GITHUB_TOKEN` with no PAT or secret configuration required
- designed to be called from any repo in the org via a single `uses:` line

## 🕹️ Usage

### Clear `Stage` field on issue close

**`clear-stage-field.yml`** is the reusable workflow. It clears the org-level `Stage` issue field when called, using a `DELETE` request authenticated with `GITHUB_TOKEN`.

**`clear-stage-field-on-issue-close.yml`** is an example calling workflow. Add this to each repo that should participate in syncing:

```yml
# SOME_ORG/SOME_REPO/.github/workflows/clear-stage-field-on-issue-close.yml
name: 🗑️ Clear 'Stage' field on issue close
run-name: "🗑️ Clear stage for closed issue: ${{ github.event.issue.title }}"
permissions:
  issues: write
on:
  issues:
    types: [closed]
jobs:
  clear-stage-field-on-close:
    name: 🗑️ Clear 'Stage' field on issue close
    uses: caret-collective/issue-field-workflows/.github/workflows/clear-stage-field.yml@main
```

## 🤖 Advanced Usage

### Updating the API version

The reusable workflow pins to a specific GitHub REST API version via `GH_API_VERSION`. When GitHub releases a new stable version, update this value in `clear-stage-field.yml`:

```yaml
env:
  GH_API_VERSION: "2026-03-10"
```

See [GitHub's API version documentation](https://docs.github.com/en/rest/about-the-rest-api/api-versions) for the latest available version.

### Updating the field ID

The `Stage` field ID is stored in `STAGE_ISSUE_FIELD_ID` in `clear-stage-field.yml`. If you fork this repo or use a different field, find the field ID by calling:

```
GET https://api.github.com/orgs/{org}/issues/fields
```

Then update the env var accordingly.

## 🛟 Support

Need help? See the [support resources](https://github.com/caret-collective/.github/blob/main/docs/SUPPORT.md) for information on how to:

- request features
- report bugs
- ask questions
- report security vulnerabilities

## 🤝 Contributing

Want to help out? Pull requests are welcome for:

- feature implementations
- bug fixes
- documentation
- tests

See the [contribution guide](../../contribute) for more details.

## 🧾 License

Copyright © 2026 [John Goodliff](https://johng.io/r/issue-field-workflows) ([@twocaretcat](https://github.com/twocaretcat)).

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
