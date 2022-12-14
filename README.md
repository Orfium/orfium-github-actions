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
    python_version: "3.10"
    environment_id: review
    environment_suffix: ${{ github.event.pull_request.number }}
    dynamic_cf_parameters: ${{ needs.build_app.outputs.json-string }}
```
* For the job name, you can leave it as shown above
* The "needs" section depends on your CI's job names and job prerequisites

For the reusable workflow's inputs, there are 4 required ones:
* python_version: The python version that will be used for workflow's commands
* environment_id: The environment_id MUST match a pre-existing environment file and a pre-existing <environment_id>_cf_parameters.json, e.g. infra/environments/production.env + infra/environments/production_cf_parameters.json + environment_id: production.
* environment_suffix: The suffix that is added to the product name ( which together form the stack name ) and some of the stack's resources as an identification. The recommendation is a) For review environments, to add the Pull Request number, b) for other environment types, to match the environment_id. Having said that, these are recommendations and anything meaningful can be added.
* dynamic_cf_parameters: (OPTIONAL) A stringified list of additional parameters that can be adedd to the sam deploy command. You can easily create such a string with a command like this:
* main_cf_template: (OPTIONAL) The default file path of the main CloudFormation template is infra/aws-deploy.yml. If there is a need for a different filename, a file in a different path in the repo or a file from an S3 URI, you can overwrite by providing here the custom template.
```
app_image=hello
test_custom_var_1=123456789012.dkr.ecr.us-east-1.amazonaws.com/test/flower:latest
test_custom_var_2=arn:aws:acm:us-east-1:123456789012:certificate/a27f06345-2000-0101-8397-8e99edd1749b

JSON_STRING=$( jq -n \
  --arg ai "$app_image" \
  --arg var1 "$test_custom_var_1" \
  --arg var2 "$test_custom_var_2" \
  '{app_image: $ai, test_custom_var_1: $var1, test_custom_var_2: $var2}' | jq '.| tostring' )
echo "json-string=$JSON_STRING" >> $GITHUB_OUTPUT -> This is the new GitHub way to output from a step/job as "set-output" will be deprecated soon.
```
and then pass it to the reusable workflow's inputs like:
```
with:
  dynamic_cf_parameters: ${{ needs.<job-name>.outputs.json-string }}
```

The workflow expects a specific folder structure and naming scheme:
* A main folder called "infra" that includes all the CloudFormation templates including the root/parent/main CloudFormation template and the samconfig.toml
* The root/parent/main CloudFormation template, must be named "aws-deploy.yml". If there is a need to not use that one, another one can be passed through the "main_cf_template" workflow input that overwrites the default value. If it's a local file, the full file path must be provided.
* A subfolder "infra/environments" that includes all the .env files and .json files that correspond with the environment_id naming scheme, plus, an optional "common.env" file for global variables across environments, like the ProductName or even aws-region if it's the same across all different environments.
* There should be only one samconfig file, called "samconfig.toml", with all the environment_types specified inside

The infra's folder structure MUST look something like this:
```
ls -lhR infra/
infra/:
total 8.0K
-rw-r--r-- 1 user user    0 Nov 10 11:45 aws-deploy.yml
drwxr-xr-x 2 user user 4.0K Nov 10 09:34 environments
-rw-r--r-- 1 user user 1.1K Nov  9 16:45 samconfig.toml

infra/environments:
total 36K
-rw-r--r-- 1 user user   31 Nov  2 18:08 common.env
-rw-r--r-- 1 user user  236 Nov  2 18:16 develop.env
-rw-r--r-- 1 user user 1.2K Nov  9 16:44 develop_cf_parameters.json
-rw-r--r-- 1 user user  236 Nov  2 18:16 prod.env
-rw-r--r-- 1 user user 1.2K Nov  9 16:44 prod_cf_parameters.json
-rw-r--r-- 1 user user  236 Nov  9 16:29 review.env
-rw-r--r-- 1 user user 1.1K Nov 10 10:56 review_cf_parameters.json
-rw-r--r-- 1 user user  236 Nov  2 18:16 staging.env
-rw-r--r-- 1 user user 1.2K Nov  9 16:44 staging_cf_parameters.json
```
