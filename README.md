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
    main_cf_template: custom_master.yml
    infra_folder_path: .
```
* For the job name, you can leave it as shown above
* The "needs" section depends on your CI's job names and job prerequisites

**P.S. The workflow assumes that the IaC related files are under the folder "infra/". If that is not the case, you need to use the workflow input: "infra_folder_path:" mentioned below, with the directory of your choice, WITHOUT the slash, e.g. '.' for root directory**

For the reusable workflow's inputs, there are 4 required ones:
* python_version: (OPTIONAL) The python version that will be used for workflow's commands
* environment_id: The environment_id MUST match a pre-existing environment file and a pre-existing <environment_id>_cf_parameters.json, e.g. environments/production.env + environments/production_cf_parameters.json + environment_id: production.
* environment_suffix: (OPTIONAL) The suffix that is added to the stack name along with the product name. The recommendation is a) For review environments, to add the Pull Request number, b) for other environment types, to match the environment_id. Having said that, these are recommendations and anything meaningful can be added. For production environments, we can leave it empty, as it will be the main environment and does not need a suffix. Moreover, it helps us make the assumption that if an environment does not have a suffix, it is the production environment and should be handled with more care than an ephemeral environment.
* dynamic_cf_parameters: (OPTIONAL) A stringified list of additional parameters that can be adedd to the sam deploy command. You can easily create such a string with a command like this:
* main_cf_template: (OPTIONAL) The default file path of the main CloudFormation template is aws-deploy.yml. If there is a need for a different filename or a file in a different path in the repo you can overwrite by providing here the custom template.
* infra_folder_path: The folder path that contains all the IaC related folders/files such as samconfig.toml, environments folder, master.yml, etc etc
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
* The root/parent/main CloudFormation template and the samconfig.toml file
* The root/parent/main CloudFormation template, must be named "aws-deploy.yml". If there is a need to not use that one, another one can be passed through the "main_cf_template" workflow input that overwrites the default value. If it's a local file, the full file path must be provided.
* A subfolder "environments" that includes all the .env files and .json files that correspond with the environment_id naming scheme, plus, an optional "common.env" file for global variables across environments, like the ProductName or even aws-region if it's the same across all different environments.
* There should be only one samconfig file, called "samconfig.toml", with all the environment_types specified inside

The infra's folder structure MUST look something like this: ( only the Infrastructure as Code files are listed )
```
ls -lhR
.:
total 8.0K
-rw-r--r-- 1 user user    0 Nov 10 11:45 aws-deploy.yml
drwxr-xr-x 2 user user 4.0K Nov 10 09:34 environments
-rw-r--r-- 1 user user 1.1K Nov  9 16:45 samconfig.toml

./environments:
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

## Cloudformation Teardown
You can add a job similar to this to your workflow for CloudFormation deployment:
```yaml
teardown-application:
    needs:
      - set-deployment-env
    uses: Orfium/orfium-github-actions/.github/workflows/teardown-reusable.yml@master
    with:
      environment_id:  ${{ needs.set-deployment-env.outputs.env-id }}
      environment_suffix: ${{ needs.set-deployment-env.outputs.env-suffix }}
```
The teardown reusable workflow handles the deletion of the cloudformations tack and all its resources. It also expects a folder structure same as the Deployment workflow above as it uses the same building blocks to find the proper stack name, AWS Account ID and IAM roles.

You will need to feed 2 required parameters to the re-usable workflow:
* environment_id: The environment_id, as mentioned above, MUST match a pre-existing environment file and a pre-existing <environment_id>_cf_parameters.json, e.g. environments/production.env + environments/production_cf_parameters.json + environment_id: production.
* environment_suffix: Unlike above, here the environment suffix is mandatory, in order to extrapolate the stack-name from the combination of the ProductName and the Environment Suffix. Because that similar method is used above to create the stack name in the first place, it used again here to reliably predict the name of the stack that we want to be deleted.

## Extract Jira Ticket ID from PR Action
This GitHub Action extracts a Jira Ticket ID either from the **PR title** or the **branch name**. It's useful for automating workflows that require a Jira Ticket ID.

### Ticket ID Format
Make sure to prefix the PR title or the branch with the following format: `[A-Z]-[0-9]`  (capital letters, dash, numbers)  
Examples:  
`PRJ-123/my-feature-branch`  
`[PRJ-123] This is the PR title`

_anything before or after the correct format is valid_

### Outputs

`jira_ticket_id`: The extracted Jira Ticket ID.  
This output can be accessed using `needs.<job_id>.outputs.jira_ticket_id`, where `<job_id>` is the ID of the job that called this action.

### Usage

You can use this action by referencing it in your workflow file with the `uses` keyword.  
Let's see a couple of examples:

```yaml
name: Get the Jira ticket from a PR

on:
    pull_request:
        types: [opened, reopened, synchronize]

jobs:
    extract-jira-ticket-id:
        uses: Orfium/orfium-github-actions/.github/workflows/extract-jira-id.yml@master
    use-jira-ticket-id:
        runs-on: ubuntu-latest
        needs: extract-jira-ticket-id
        steps:
            - name: Use the Jira ticket id
              run: echo "The Jira ticket id is ${{ needs.extract-jira-ticket-id.outputs.jira_ticket_id }}"
```