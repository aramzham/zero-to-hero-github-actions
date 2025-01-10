# github-actions-dotnet

This is a starter project for .NET that will be used as a starter for creating GitHub Actions

GitHub's CI/CD = GitHub Actions

1. you have your raw source code files on your machine
2. you push them to GitHub through a commit
3. a GitHub action runner (a virtual machine that runs your code) is triggered
4. your code is pushed to destination server

CI builds your code and creates the so called artifacts

CI means you integrate your code with others' code

CD - continuous delivery, means that you've automated the code reaching to production environment (it may be at most 1 button push)

CD - continuous deployment, if everything is green (acceptance tests, smoke tests etc.) then it goes all the way to production

find the events that trigger workflows here
https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows

give all jobs some names or it will default to ids
build:
name: "Sample build"

Infrastructure as code = we don't click around some buttons but create a sort of a configuration file where we describe what we want

Workflow = everything that's in the yaml file
Triggers = kicks off the workflow
Jobs = sequence of steps, they run parallelly
Steps = individual tasks that run
Runners = jobs need some sort of VMs to run on

on:
- push
this is the same as
on: [push]

practice GitHub actions on https://github-actions-hero.vercel.app/

env:
AZURE_WEBAPP_NAME: my-app-name
setion contains variables that will be reused in multiple jobs like this ${{ env.AZURE_WEBAPP_NAME }}

you'd like to have a PR verify workflow so at least your project builds after the PR is merged

some tips on repository settings:
- allow only squash merge
- check Always suggest updating pull request branches
- check Allow auto-merge, means if the checks pass automatically merge the request
- check Automatically delete head branches, delete the branch that opened the PR, your feature branch

tips on rulesets:
- set it on default or by pattern
- restrict deletion
- require linear history
- require a pull request before merging
    - dismiss stale pull request approvals when new commits are pushed (push a new commit when it was approved by someone => reset their vote)
    - require conversation resolution before merging (resolve all comments)
- require branches to be up to date before merging
- block force pushes

use .editorconfig to apply code styling and formatting
use "dotnet format --verbosity detailed" to format the code according to .editorconfig file

one thing to notice is when you've already implied some mandatory checks which contain already building and testing your code then you can even not specify them in other yaml workflows.
These checks may be bypassed for some users so in many cases it's fine to double check with tests.

GitHub scheduled actions run at most every 5 minutes, you can't run them more often, you don't even need it (GitHub may even drop your jobs)

a use case for cron workflows maybe security jobs once a week for vulnerable packages, vulnerable docker images

env:
API_CSPROJ_PATH: ./src/GitHubActionsDotNet.Api/GitHubActionsDotNet.Api.csproj
these variables are used for non-secret things

env: can also be job specific so you don't share across all the steps

GitHub will mask the secrets even if you try to log them

Azure AppService is optimized to run web applications, it basically takes your dlls, exes or docker containers and runs them

Azure subscriptions - per team, per environment (internal team-dev, internal team-prod)
Resource group - per app, per environment (app1-dev can have sql db, azure functions etc., app2-dev just needs a db)

azure resource naming conventions - https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming

azure abbreviation recommendations - https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations

pick Premium V3 pricing plan for app service for deployment slots for 0 downtime deployments

you have to upload and download artifacts between jobs because jobs cannot share the file system

      - uses: actions/download-artifact@v4
        with:
          name: my-artifact
          path: artifacts2/
this says - download the files by id "my-artifact" and put them into artifacts2 folder

deploy_dev:
name: Deploy dev
runs-on: ubuntu-latest
needs: build
means deploy_dev will wait for build to finish

website contributor role in azure means that it can upload files into app service

to deploy from GitHub actions to azure you'd need
- a managed identity,
- assign it a website contributor role,
- set the ClientID, tenantId, subscriptionId in repository secrets,
- create a federated login for GitHub in azure managed identity
- add id-token: write permission in yaml file


if you want to create a reusable step across different repositories
then you need to specify the organization/repo and @branch at the end
uses: aramzham/my-repo-name/.github/workflows/step-deploy.yml@main
you can also use a file version name step-azure-app-service-deploy-v1.yml and not specify the branch

what you should have in CI:
- restore packages
- compile
- test
- format
- linting (SonarQube)
- security scans (CodeQL)
- upload artifacts
- alerting on failure

what you should have in CD:
- download artifacts
- deploy artifacts
- zero downtime (slot swaps)
- deploy IAC
- smoke tests
- alerting on failure

main.bicep is the entry point to your resources
