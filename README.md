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
  issue labeled `very-audit`. It runs in your Actions and reports findings on the issue.
- **Security and dependency checks.** Weekly checks of repository settings, Dependabot
  alerts and outdated dependencies, using GitHub's API.

## What it can and can't do

| very-bot can | very-bot can't |
|---|---|
| Read code and repository settings | Push code or merge pull requests |
| Label pull requests | Run, edit or approve your workflows or deployments |
| Open and comment on issues | Read your secrets |

## Where your code goes

Reviews and audits run on **your** runners. When AI is used, your code is sent to
Anthropic's API, under a short-lived credential scoped to your organization, and to no Very
system. Very receives only the findings. No API key is ever stored in your repositories.

You can uninstall very-bot at any time from your organization's settings.

## Adding reviews to a repository

Your Very engineer adds this file through a pull request. The four IDs are identifiers,
not secrets; they come from Very's setup for your organization.

```yaml
# .github/workflows/very-review.yml
name: very-bot review
on:
  pull_request:
    types: [labeled]
jobs:
  review:
    if: github.event.label.name == 'very-review'
    uses: verypossible-labs/very-bot-workflows/.github/workflows/review.yml@v1
    permissions:
      contents: read
      pull-requests: write
      id-token: write
    with:
      federation-rule-id: fdrl_…
      organization-id: …
      service-account-id: svac_…
      workspace-id: wrkspc_…
```
