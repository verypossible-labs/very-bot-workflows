# very-bot

very-bot is Very Technology's engineering assistant for the GitHub organizations we build
software with. It's a GitHub App, and only active Very clients can keep it installed: any
other installation is removed automatically.

This repository holds the GitHub Actions workflows very-bot runs in your repositories.
They're public so your repositories can call them; nothing else about very-bot lives here.

## What it does

- **AI code reviews.** Adding the `very-review` label to a pull request runs a review in
  your own GitHub Actions and posts comments. It never approves or merges. Remove and
  re-add the label to review again.
- **Standards audits.** A Very engineer can request an audit, which very-bot opens as an
  issue labeled `very-audit`. It runs in your Actions, changes nothing, and reports
  findings on the issue. The issue stays open while a check fails; the next audit
  replaces it. Only very-bot's own issues start an audit.
- **Security and dependency checks.** Weekly checks of repository settings, Dependabot
  alerts and outdated dependencies, using GitHub's API.

## What it can and can't do

| very-bot can | very-bot can't |
|---|---|
| Read code and repository settings | Push code or merge pull requests |
| Label pull requests | Run, edit or approve your workflows or deployments |
| Open and comment on issues | Read your secrets |

## Where your code goes

Reviews and audits run on **your** runners. When AI is used, your code is sent only to the
AI provider agreed with you for your organization, under a short-lived credential scoped
to your organization, and to no Very system. Very receives only the findings. No API key
is ever stored in your repositories.

You can uninstall very-bot at any time from your organization's settings.

## Adding very-bot to a repository

An organization owner sets four organization variables once, under **Settings → Secrets
and variables → Actions → Variables**. They are identifiers, not secrets, and come from
Very's setup for your organization:

| Variable | Looks like |
|---|---|
| `VERY_BOT_FEDERATION_RULE_ID` | `fdrl_…` |
| `VERY_BOT_ORGANIZATION_ID` | a UUID |
| `VERY_BOT_SERVICE_ACCOUNT_ID` | `svac_…` |
| `VERY_BOT_WORKSPACE_ID` | `wrkspc_…` |

Then each repository gets this file through a pull request. It's the same in every
repository, and replaces an older `very-review.yml`: delete that in the same pull
request, or each label runs two reviews.

```yaml
# .github/workflows/very-bot.yml
name: very-bot
on:
  pull_request:
    types: [labeled]
  issues:
    types: [labeled]
jobs:
  review:
    if: github.event_name == 'pull_request' && github.event.label.name == 'very-review'
    uses: verypossible-labs/very-bot-workflows/.github/workflows/review.yml@v1
    permissions:
      contents: read
      pull-requests: write
      id-token: write
    with:
      federation-rule-id: ${{ vars.VERY_BOT_FEDERATION_RULE_ID }}
      organization-id: ${{ vars.VERY_BOT_ORGANIZATION_ID }}
      service-account-id: ${{ vars.VERY_BOT_SERVICE_ACCOUNT_ID }}
      workspace-id: ${{ vars.VERY_BOT_WORKSPACE_ID }}
  audit:
    if: github.event_name == 'issues' && github.event.label.name == 'very-audit'
    uses: verypossible-labs/very-bot-workflows/.github/workflows/audit.yml@v1
    permissions:
      contents: read
      issues: write
      id-token: write
    with:
      federation-rule-id: ${{ vars.VERY_BOT_FEDERATION_RULE_ID }}
      organization-id: ${{ vars.VERY_BOT_ORGANIZATION_ID }}
      service-account-id: ${{ vars.VERY_BOT_SERVICE_ACCOUNT_ID }}
      workspace-id: ${{ vars.VERY_BOT_WORKSPACE_ID }}
```

## Versions

`@v1` always points at the latest v1 release, so fixes reach you without a PR; nothing
breaking ever lands on it. To freeze on one release instead, pin its full version
(`@v1.1.0`) or commit SHA. Releases are listed on this repository's Releases page.
