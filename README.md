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


## Trigger Release to Production PR (scheduled reusable workflow)

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
If the following requirements are met:
* no PR between `main` and `develop` is open at the time of the workflow run
* there is diff between the two branches

a `Release to Production` pr will be opened every Tuesdays and Thursdays at 07:25AM UTC, with `develop` as
head branch and `main` as the base one (main <-- develop), with labels `release` and `automated pr` and with 
reviewers GitHub users with username `username1` and `username2` and team with slug `teamA`.

## Cloudformation Deployment
You can add a job similar to this to your workflow for CloudFormation deployment:
```yaml
deploy:
  name: ${{ github.event.repository.name }} Deployment
  needs:
    - build_app
  uses: Orfium/orfium-github-actions/.github/workflows/deploy.yml@master
  with:
    app_image: ${{ needs.build_app.outputs.app_image }}
    environment_type: review
    python_version: "3.10"
    InstanceType: "t3.xlarge"
    DBInstanceClass: "db.t3.medium"
```
* For the name, you can leave it as shown above
* The "needs" section depends on your CI's job names and job prerequisites

For the reusable workflow's inputs, there are 3 required ones some optional ones. The required are:
* app_image: This will be the application's image that is built in a previous step. It needs to be outputed from that job and be set as an input here. Documentation: https://docs.github.com/en/actions/using-jobs/defining-outputs-for-jobs
* environment_type: Inside the workflow itself there is a list of allowed values that MUST match a pre-existing environment file, e.g. production.env + environment_type: production. If there is a need for additional options, both a new entry in the input's "options" list and the corresponding file must be added.
* python_version: The python version that will be used for workflow's commands

As for the optional inputs, all of them can be found in the wokrflow itself. An example is shown above with InstanceType and DBInstanceClass. The list can be extended with more optional inputs. The handling of them ( or the handling of the absense of them ) can be found in the "Aggregate Parameter Overrides" step in the workflow.

The workflow expects a specific folder structure and naming scheme:
* A main folder called "infra" that includes all the CloudFormation templates, the samconfig.toml and an optional parameters.json file ( more on that later )
* A subfolder "infra/environments" that includes all the .env files that correspond with the environment_type naming scheme, plus, a "common.env" for global variables across environments, like the ProductName
* The root/parent/main CloudFormation template, must be named "aws-deploy.yml"
* There should be only one samconfig file, called "samconfig.toml", with all the environment_types specified inside

There is support for both parameter_overrides inside the samconfig.toml file and additional parameters, the workflow's optional inputs. Moreover, an external file called parameters.json can be optionally utilized, that may contain additional parameters that we want to pass to the deployment. All of them are combined and passed to sam deploy.
