# register-deregister-sso-callback-url.yml

This GitHub Action assumes an AWS IAM role named [GithubSSORole](https://github.com/Orfium/aws-management/pull/889/files) and registers or deregisters a URL from a client.
It takes a client ID and a URL as inputs and uses AWS CLI to assume the role and invoke the appropriate API actions to register or deregister the URL from the client.

## Inputs

### `client_id`
Required The client ID of the client to register or deregister the URL from.

### `url`
Required The URL to register or deregister from the client.

## Example Usage

```
jobs:
  test_register:
    runs-on: ubuntu-latest
    uses: Orfium/github-actions/.github/workflows/register-deregister-sso-callback-url.yml@master
    with:
      client_id: 'ylaDRCJOZtXxXhzdMKHMgosLikD28WQA'
      url: 'https://two.orfium.com'
      action: 'register'

  test_deregister:
    runs-on: ubuntu-latest
    uses: Orfium/github-actions/.github/workflows/register-deregister-sso-callback-url.yml@master
    with:
      client_id: 'ylaDRCJOZtXxXhzdMKHMgosLikD28WQA'
      url: 'https://two.orfium.com'
      action: 'deregister'
```
---

# register-deploy-to-getdx.yml
This GitHub Action is used to register the deployments (successful or failed) of a service in the [DX Data Cloud](https://app.getdx.com/datacloud/reports/deploys?teams=&tags=&range=l4w&interval=w&services=&envs=).  

The allowed branches that a deployement should be registered from are: _main, master, staging, develop, development._

## Inputs

### `service`
The name of the service to be deployed.

## Example Usage

This action should be used as a separate workflow which is triggered by a "deployment" workflow.

```
name: Register deploy in DX Data Cloud
on:
  workflow_run:
    workflows:
      - CI/CD
    types:
      - completed
    branches:
      - main
      - staging
jobs:
  register_deploy:
    uses: Orfium/github-actions/.github/workflows/register-deploy-to-getdx.yml@master
    with:
        service: "<Name of the service>"
    secrets:
      DX_DEPLOYMENT_API_TOKEN: ${{ secrets.DX_DEPLOYMENT_API_TOKEN }}

```
In the example above, "CI/CD" is the name of the workflow that performs the deployment.  
`DX_DEPLOYMENT_API_TOKEN` secret should be available to the caller's repository.