---
layout: post
title: Enforcing Reusable Workflows for Standardization
date: '2021-11-02 01:22:01'
image: /assets/images/2021/11/compliance-reusable/something.jpg
description: >
  Reusable workflows are great, but how do you ensure that teams are using your reusable workflows? In this post I show how you can structure repos, teams and environments to ensure standardization for your workflows.
tags:
- build
- security
---

1. TOC
{:toc}

# Problem Statement

Imagine you want to ensure that code that gets deployed is scanned using CodeQL. Furthermore, you want to enforce a specific set of steps for deploying your apps.

For sake of brevity, let's imagine two Teams: the App Team and the Compliance Team. Let's take a look at the roles of these two teams:

Item|App Team|Compliance Team
--|--|--
`main` branch|Ensure that all code changes to `main` are peer-reviewed and pass quality gates before merge|Same as App Team
Build, test and package the application|Responsible for this workflow|Reviews, but is really interested in the final deployable package
Code Scan|-|Must ensure that all deployable code has been scanned
Deployment|Provide package for deployment. Can also collaborate on deployment steps.|Final accountability and maintenance of deployment process
Dev Environment|May want to gate with manual approval|Just supply the deployment process
Prod Environment|-|Ensure only code from protected branches is deployed and configure manual approval gate

One way to achieve this may be for all PRs to require approvals from the Compliance Team - but this is not practical and does not scale.

A better solution is to use the following mechanism:
1. Compliance Team configures the repo to allow `admin` access for the Compliance Team and `write` access for the App team.
1. Compliance Team creates reusable workflows for code scanning and deployment in a Workflows repo.
1. Compliance Team creates one or more `compliant-` workflows in the app repo's `.github/workflows` directory. These workflows call the reusable workflows in the Worfklows repo.
1. Compliance Team creates a `CODEOWNERS` file, enforcing that changes to `.gitub/workflows/compliant-*` require approvals from the Compliance Team
1. Compliance Team configures `main` branch to be protected. `Require Code Review by CODEOWNERS` is enabled. `Required Checks` is enabled, requiring the `compliant-code-scan` workflow to pass before allowing merges. Other settings can be set in collaboration with the App Team.
1. Compliance Team creates `dev` and `prod` environments, requiring the appropriate approvers. On `prod`, require appropriate approvers and also require that only protected branches can be deployed. 

With this set of configuration, the Compliance Team can ensure that teams are following approved processes. Let's walk through this configuration step by step. 

# Configuration

## Team and Repo Configuration

Let's start by configuring the Teams. While you could configure these setting using individuals, setting the `CODEOWNERS` to be owned bu the Compliance Team is more scalable - and as folks leave/join the team, you don't have to update configuration in app repos. So you'll need to create at least the Compliance Team in your organization, adding appropriate members. You can even make the team `Secret` if you choose to:

![Create a Compliance Team](/assets/images/2021/11/compliance-reusable/create-compliance-team.png){: .center-image }

Create a Compliance Team.
{:.figcaption}

Now you can create a repo for your approved workflows. Obviously the Compliance Team should be the owners/contributors to this repo. Other teams can create PRs if they wish to, but should not be able to directly write to this repo, or at leat to the `main` branch.

For this example, I'm putting my reusable workflows into a repo called `super-approved-workflows`.

You can now create an App Team, though this is not necessary.

Now you can create the application repo. An administrator should ensure that the Compliance Team is set with `admin` priveledges on this repo, since they'll have to do some initial configuration and will be adding files that the App Team cannot change without Compliance Team approvals. In the settings tab of my `compliant-app` repo, I've configured the Teams like this:

![Configure Team access on app repo](/assets/images/2021/11/compliance-reusable/app-repo-settings.png){: .center-image }

CConfigure Team access on app repo.
{:.figcaption}

> Note: Ensure that the App Team are not given `admin` permissions, or they will be able to work around the compliance settings! I think that the team should only require `write` permissions, but there may be cases where `maintain` is required. Default to lowest priviledges first (i.e. `write`) and create an App Maintainer team for a subset of the app team if you really need `maintain` permissions for some operations. You can see the different between `maintain` and `write` [here](https://docs.github.com/en/organizations/managing-access-to-your-organizations-repositories/repository-roles-for-an-organization#permissions-for-each-role).

## `CODEOWNERS` and `compliant-` Workflows

Next, the Compliance Team created a `CODEOWNERS` file in the `.github` folder of the app repo. This tells the repo that any changes to the files specified require review by the Compliance Team:

~~~yml
{% raw %}
# file: '.github/CODEOWNERS'
# Changes to `compliant-` workflows requires @compliance-team approval
/.github/workflows/compliant-* @colinsalmcorner/compliance-team
{% endraw %}
~~~

`CODEOWNERS` file to enforce Compliance Team approvals for changes to `compliant-*` workflows.
{:.figcaption}

> Note: The `compliant-` prefix is an arbitrary prefix. It can be anything you want, but in this case I wanted to distinguish between workflows the App Team can mess with and those that they can't. All workflows must be in the `.github/workflows` folder, so adding a prefix was the only way this would work.

### App Team Code Scan Workflow

Now the Compliance Team can add a workflow to scan code. We'll have a look at the called workflow later, but for now, here is the workflow for the App Team:

~~~yml
{% raw %}
# file: '.github/workflows/compliant-scan.yml'
name: Scan app

on:
  workflow_dispatch:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]

jobs:
  scan:
    name: Code scan
    uses: colinsalmcorner/super-approved-workflows/.github/workflows/codeql-scan.yml@main
    with:
      languages: '["csharp"]'
{% endraw %}
~~~

`compliant-scan.yml` workflow that invokes the centrally managed Code Scanning workflow.
{:.figcaption}

Notes:
1. The file name has the special `compliant-` prefix. Any changes to this file will require Compliance Team approvals (configured via the `CODEOWNERS` file).
1. We can add whatever triggers make sense, but should at least have `pull_request` and `push` to `main` trigger this workflow.
1. The job just calls the centrally managed, reusable code scan workflow, passing in the language(s) we want scanned.

### App Team Code Scan Workflow

Now the Compliance Team can add a workflow to scan code. We'll have a look at the called workflow later, but for now, here is the workflow for the App Team:

~~~yml
{% raw %}
# file: '.github/workflows/compliant-deploy.yml'
name: Deploy Pipeline

on:
  workflow_dispatch:

jobs:
  build:
    name: Build
    uses: colinsalmcorner/compliant-app/.github/workflows/build.yml@main
    with:
      artifact-name: drop

  dev:
    name: Deploy to DEV
    needs: build
    uses: colinsalmcorner/super-approved-workflows/.github/workflows/deploy-app.yml@main
    with:
      artifact-name: drop
      environment-name: dev
      environment-url: https://dev.my-super-app.net
    secrets:
      PASSWORD: ${{ secrets.DEV-PASSWORD }}
  
  prod:
    name: Deploy to PROD
    needs: build
    uses: colinsalmcorner/super-approved-workflows/.github/workflows/deploy-app.yml@main
    with:
      artifact-name: drop
      environment-name: prod
      environment-url: https://my-super-app.net
    secrets:
      PASSWORD: ${{ secrets.PROD-PASSWORD }}
{% endraw %}
~~~

`compliant-deploy.yml` workflow that invokes a "local" reusable workflow and then invokes the centrally managed Deploy workflow for each environment.
{:.figcaption}

Notes:
1. Again, the file name has the special `compliant-` prefix. Any changes to this file will require Compliance Team approvals (configured via the `CODEOWNERS` file).
1. We can add whatever triggers make sense - in this case, I've just configured manual trigger (via `workflow_dispatch`).
1. There are three jobs: `build`, `dev` and `prod`.
1. The `build` job is calling a "local" (in the same repo) reusable workflow that the App Team has full control over that builds, tests and packages the app. To make this work with the deployment workflows, we need to publish an artifact (called `drop` in this case). The deploy workflow will download the artifact and then deploy.
1. We can pass whatever parameters we need to here - in this case I've configured parameters for the `artifact-name`, `environment-name` and `environment-url`.
1. Secrets are tricky since reusable workflows can't (yet?) read environment secrets. So you have to specify secrets on the repo (or org) level that are prefixed with the environment name in some way.

## The Reusable Workflows

Now back in the Workflows repo, the Compliance Team can add the reusable workflows. In this example, we're going to add the Code Scanning workflow and a deployment workflow.



# Conclusion

Code Quality Metrics are useful, but their criticality typically decreases over time, especially when teams implement good quality gates in their software development life cycle. The criticality of Code Security, on the other hand, steadily increases over time as code bases and attack vectors grow.

While there is a lot of tooling in both the Code Quality Metrics and Code Security spaces, GitHub Advanced Security offers a unique platform that enables developer-first security, integrating security into developer workflows naturally and seamlessly, making it an indispensable tool for modern software development.

Happy securing!