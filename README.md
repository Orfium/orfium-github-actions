# orfium-github-actions
Reusable GitHub actions


## FE Combine Dependabot PRs

Add this to your project's workflows.

```yaml
name: 'Combine PRs'

on:
  workflow_dispatch:
    inputs:
      branchPrefix:
        description: 'Branch prefix to find combinable PRs based on'
        required: true
        default: 'dependabot'
      combineBranchName:
        description: 'Name of the branch to combine PRs into'
        required: true
        default: 'combined-dependabot'
      ignoreLabel:
        description: 'Exclude PRs with this label'
        required: true
        default: 'nocombine'

jobs:
  call-workflow:
    uses: Orfium/orfium-github-actions/.github/workflows/fe-combine-dependabot-prs.yml@master
    with:
      branchPrefix: github.event.inputs.branchPrefix
      combineBranchName: github.event.inputs.combineBranchName
      ignoreLabel: github.event.inputs.ignoreLabel
    secrets:
      token: ${{ secrets.DEPENDABOT_ACCESS_TOKEN }}


```

On workflows, select the Combine PRs workflow and simply press run.


## Trigger release pr to production (periodically)

By adding this to your project:
```yaml
name: 'Trigger Release to Production PR'
on:
  schedule:
    # Every Tuesdays (2) and Thursdays (4) at 07:25 UTC
    - cron: "25 7 * * 2,4"

jobs:
  call-workflow:
    uses: Orfium/orfium-github-actions/.github/workflows/trigger-release-pr.yml@master
    with:
      baseBranchName: 'main'
      headBranchName: 'develop'
      releasePrLabels: 'release,automated pr'
      releasePrIsDraft: false
      releasePrReviewers: 'username1,username2'
      releasePrTeamReviewers: 'teamA'
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```
You can trigger periodically (using cron schedule) the creation of a pr (e.g. main <- develop).
