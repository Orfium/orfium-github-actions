# How to contribute

## Getting Started
* Create a Jira ticket with your request if one does not already exist.
  * Clearly describe the request.
  * Since some changes in the actions may be breaking ( which may result in CI/CD workflows failing ), if you know it beforehand, please add it to the ticket.
* Make sure that all Actionlint issues are fixed
* If you are not sure how and where to start contact the DevOps team.

## Making Changes
* Clone this repository locally
* Create a feature branch from the `master` branch
  * To quickly create a feature branch based on `master`, run `git checkout -b JIRA-TICKET/my_contribution`
* Make commits of logical and atomic units
* Test a workflow with a test repository that has a CI/CD workflow that is using the changes GitHub reusable workflow. It's also best to test first with an unchanged CI/CD to check if the changes are backwards compatible

## Submitting Changes
* Push your changes to a feature branch in the repository
* Create a pull request from your feature branch to the `master` branch
* Use the related Jira ticket number in the PR title or body to link your PR with the Jira ticket
* The DevOps team will be automatically notified to review your Pull Request
* Two approvals minimum are needed before the PR can be merged
* After feedback has been given and your PR is approved it will/can be merged to `master`