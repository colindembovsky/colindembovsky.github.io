---
layout: post
title: Enforcing Reusable Workflows for Standardization
date: '2021-11-03 01:22:01'
image: /assets/images/2021/11/compliance-reusable/hacker.jpg
description: >
  Reusable workflows are great, but how do you ensure that teams are using your reusable workflows? In this post I show how you can structure repos, teams and environments to ensure standardization for your workflows.
tags:
- build
- security
---

1. TOC
{:toc}

> Image from [www.freepik.com](https://www.freepik.com/vectors/flyer)

# Problem Statement

There is a delicate balance between _team autonomy_ and _enterprise alignment_. Too much autonomy can result in chaos, rework and runaway spending. Too much red tape can result in logn cycle times, frustration and lack of innovation. But it is possible to implement some level of compliance and leave teams some autonomy too.

Imagine you want to ensure that code that gets deployed is scanned using CodeQL. Furthermore, you want to enforce a specific set of steps for deploying your apps. You would prefer your app teams to be able to build, test and package applications themselves. In this post I'll show how you can achieve do this using GitHub.

We'll be working with two imaginary Teams: an App Team and a Compliance Team. Let's take a look at the roles of these two teams:

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

With this set of configuration, the Compliance Team can ensure that teams are following approved processes, not beome a bottleneck, and leave the App Team to get on with their work. Let's walk through this configuration step by step.

# Configuration

> Note: All the code for this demo is available in this [app repo](https://github.com/colinsalmcorner/compliant-app) and in this [workflow repo](https://github.com/colinsalmcorner/super-approved-workflows). I've made these public repos to share content, but typically these would be internal or private repos in your GitHub org.

## Team and Repo Configuration

Let's start by configuring the Teams. While you could configure these setting using individuals, setting the `CODEOWNERS` to be owned bu the Compliance Team is more scalable - and as folks leave/join the team, you don't have to update configuration in app repos. So you'll need to create at least the Compliance Team in your organization, adding appropriate members. You can even make the team `Secret` if you choose to:

![Create a Compliance Team](/assets/images/2021/11/compliance-reusable/create-compliance-team.png){: .center-image }

Create a Compliance Team.
{:.figcaption}

Now you can create a repo for your approved workflows. Obviously the Compliance Team should be the owners/contributors to this repo. Other teams can create PRs if they wish to, but should not be able to directly write to this repo, or at least to the `main` branch.

For this example, I'm putting my reusable workflows into a repo called `super-approved-workflows`.

You can now create an App Team, though this is not necessary.

Now you can create the application repo. An administrator should ensure that the Compliance Team is set with `admin` priveledges on this repo, since they'll have to do some initial configuration and will be adding files that the App Team cannot change without Compliance Team approvals. In the settings tab of my `compliant-app` repo, I've configured the Teams like this:

![Configure Team access on app repo](/assets/images/2021/11/compliance-reusable/app-repo-settings.png){: .center-image }

CConfigure Team access on app repo.
{:.figcaption}

> Note: Ensure that the App Team are not given `admin` permissions, or they will be able to work around the compliance settings! I think that the team should only require `write` permissions, but there may be cases where `maintain` is required. Default to lowest priviledges first (i.e. `write`) and create an App Maintainer team for a subset of the app team if you really need `maintain` permissions for some operations. You can see the different between `maintain` and `write` [here](https://docs.github.com/en/organizations/managing-access-to-your-organizations-repositories/repository-roles-for-an-organization#permissions-for-each-role).

## Workflows

To help visualize how the workflow files are organized, I drew this awesome PowerPoint art:

![Overview of how the workflows are structured](/assets/images/2021/11/compliance-reusable/workflows-for-compliance.png){: .center-image }

Overview of how the workflows are structured.
{:.figcaption}

Let's now dig into the workflows.

### App Team Code Scan Workflow

The Compliance Team should add a workflow in the App Repo to scan code. We'll have a look at the called workflow later, but for now, here is the workflow for the App Team:

~~~yml
{% raw %}
# file: 'APP_REPO/.github/workflows/compliant-scan.yml'
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

The Compliance Team should now add a workflow in the App Repo to scan code. We'll have a look at the called workflow later, but for now, here is the workflow for the App Team:

~~~yml
{% raw %}
# file: 'APP_REPO/.github/workflows/compliant-deploy.yml'
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
      PASSWORD: ${{ secrets.DEV_PASSWORD }}
  
  prod:
    name: Deploy to PROD
    needs: dev
    uses: colinsalmcorner/super-approved-workflows/.github/workflows/deploy-app.yml@main
    with:
      artifact-name: drop
      environment-name: prod
      environment-url: https://my-super-app.net
    secrets:
      PASSWORD: ${{ secrets.PROD_PASSWORD }}
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

### The Build Workflow

Before we switch over to the reusable workflows from the Compliance Team, let's have a look at the `build.yml` workflow from the App Team:

~~~yml
{% raw %}
# file: 'APP_REPO/.github/workflows/compliant-deploy.yml'
name: Build app

on:
  workflow_call:
    inputs:
      artifact-name:
        description: Name of the artifact to upload
        type: string
        default: drop

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 5.0.x
    - name: Restore dependencies
      run: dotnet restore
    - name: Build
      run: dotnet build --no-restore
    - name: Test
      if: ${{ inputs.test }}
      run: dotnet test --no-build --verbosity normal
    - name: Publish
      run: dotnet publish -c Release -o tmpdrop
    - name: Upload a Build Artifact
      uses: actions/upload-artifact@v2.2.2
      with:
        name: ${{ inputs.artifact-name }}
        path: tmpdrop/**
        if-no-files-found: error
{% endraw %}
~~~

`compliant-deploy.yml` workflow that invokes a "local" reusable workflow and then invokes the centrally managed Deploy workflow for each environment.
{:.figcaption}

### Reusable Code Scan Workflow

This is just a stock CodeQL workflow that the Compliance Team will create in the Compliance repo. Obviously it must be reusable:

~~~yml
{% raw %}
# file: 'COMPLIANCE-REPO/.github/workflows/codeql-scan.yml'
name: "CodeQL"

on: 
  workflow_call:
    inputs:
      languages:
        description: Languages to scan, in the format of JSON array, e.g. '["csharp", "typescript"]'
        required: true
        type: string
  
jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest

    permissions:
      actions: read
      contents: read
      security-events: write

    strategy:
      fail-fast: false
      matrix:
        language: ${{ fromJSON(inputs.languages) }}

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Initialize CodeQL
      uses: github/codeql-action/init@v1
      with:
        languages: ${{ matrix.language }}
    
    - name: Autobuild
      uses: github/codeql-action/autobuild@v1

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v1
{% endraw %}
~~~

`codeql-scan.yml` workflow that runs CodeQL scanning.
{:.figcaption}

Notes:
1. The `workflow_call` makes this workflow reusable.
1. We pass in a JSON string of languages (since we can't pass arrays to reusable workflows).
1. We specify minimal `permissions` for the workflow.
1. We create a matrix that spawns a job for each `language`: checkout, initialize, autobuild and then analyze.

### Reusable Deploy Workflow

This example shows how to download the build artifact and then dumps the secret to show that it's getting a value. The actual deployment steps would be inserted into this workflow in real life.

~~~yml
{% raw %}
# file: 'COMPLIANCE-REPO/.github/workflows/deploy-app.yml'
name: Build dotnet application

on:
  workflow_call:
    inputs:
      runs-on:
        description: Platform to execute on
        type: string
        default: ubuntu-latest
        
      artifact-name:
        description: Name of the artifact to deploy
        type: string
        default: drop
      
      environment-name:
        description: Name of environment
        type: string
        required: true
      
      environment-url:
        description: URL of environment
        type: string
        required: true
    
    secrets:
      PASSWORD:
        required: true

jobs:
  deploy:
    name: Deploy app
    
    runs-on: ${{ inputs.runs-on }}
    
    environment:
      name: ${{ inputs.environment-name }}
      url: ${{ inputs.environment-url }}
    
    steps:
      - uses: actions/checkout@v2

      - uses: actions/download-artifact@v2
        with:
          name: ${{ inputs.artifact-name }}

      - name: Display structure of downloaded files
        run: ls -R

      - name: Password
        run: echo "JUST FOR TESTING: Password is ${{ secrets.PASSWORD }}"

      ## INSERT deployment steps here
{% endraw %}
~~~

`deploy-app.yml` workflow that downloads an artifact and has a placeholder for "real" deployment steps.
{:.figcaption}

Notes:
1. The `workflow_call` makes this workflow reusable.
1. We specify the input args and secrets that are required for deployment steps.
1. The `runs-on` and `environment` settings are configurable.
1. The workflow downloads the artifact, but doesn't actually do anything with it - the real deployment steps would go at the bottom of the job.

## `CODEOWNERS` File

The Compliance Team now creates a `CODEOWNERS` file in the `.github` folder of the App repo. This tells the repo that any changes to the files specified require review by the Compliance Team:

~~~yml
{% raw %}
# Changes to `compliant-` workflows requires @compliance-team approval
/.github/workflows/compliant-* @colinsalmcorner/compliance-team

# Changes to `CODEOWNERS` requires @compliance-team approval
/.github/CODEOWNERS @colinsalmcorner/compliance-team
{% endraw %}
~~~

`CODEOWNERS` file to enforce Compliance Team approvals for changes to `compliant-*` workflows.
{:.figcaption}

> Note: The `compliant-` prefix is an arbitrary prefix. It can be anything you want, but in this case I wanted to distinguish between workflows the App Team can mess with and those that they can't. All workflows must be in the `.github/workflows` folder, so adding a prefix was the only way this would work.

## Branch Protection Policy

Now that we have the scaffolding in place, we need to ensure that no-one who doesn't have permissions overwrites or changes files that they shouldn't! We can do that using branch protection policies.

In the App Repo, navigate to **Settings->Branches** and apply the following settings to the `main` branch (or whatever your protected branch is called):

![Configuring Branch Protection](/assets/images/2021/11/compliance-reusable/branch-protection-settings.png){: .center-image }

Configuring Branch Protection.
{:.figcaption}

Notes:
1. We enable `Require a pull request before merging` and `Require approvals` - these should be default on any repo, regardless of compliance level.
1. We enable `Require review from Code Owners` to ensure the Compliance Team is notified of changes to the `compliant-` workflows.
1. To ensure that Code Scanning is performed for all deployable code, we enable `Require status checks to pass before merging` and then select the `Code Scan` workflow. We also enable `Require branches be up to date before merging` as a good practice.

## Environments

The final bit of configuration is performed on the Environments. Let's look at the settings for the `prod` environment:

![Configuring the Prod Environment](/assets/images/2021/11/compliance-reusable/environment-config.png){: .center-image }

Configuring the Prod Environment.
{:.figcaption}

Notes:
1. We add appropriate manual approvals.
1. Under `Deployent branches` we configure only `Protected branches` may be deployed (currently only `main`)

## Secrets

As mentioned, we'll have to create environment-prefixed repo (or org) secrets since reusable workflows don't support environment secrets. The Compliance Team can create `DEV_PASSWORD` and `PROD_PASSWORD` in this example.

# Working Like a Charm

Updates to `main` and PRs to `main` now trigger the code scanning workflow:

![Code Scan running](/assets/images/2021/11/compliance-reusable/running-code-scan.png){: .center-image }

Code Scan running.
{:.figcaption}

If the Deploy pipeline is triggered in the Actions tab, the pipeline executes as expected. First, the `build` job builds, tests and packages the application. Then the `dev` deployment job is triggered to deploy to the `dev` environment. Finally, the `prod` job is triggered, but only from the `main` branch and with the pause for manual approval:

![Deploy Pipeline running](/assets/images/2021/11/compliance-reusable/running-deploy.png){: .center-image }

Deploy Pipeline running.
{:.figcaption}

# Can It Be Bypassed?

Now that we have things configured, let's see if we can bypass anything! I created an account called `faux-colin` that is a contributor on the App Repo - he's part of the App Team. Let's imagine he's nefarious too!

## Attempt to Inject Bad Code

`faux-colin`'s first attack vector might be the code itself. So he tries to change the code on `main`. Cloning locally, changing the code, and pushing fails. In the UI, there's no way to change code other than creating a branch and submitting a PR:

![Trying to inject bad code into main](/assets/images/2021/11/compliance-reusable/inject-bad-code.png){: .center-image }

Trying to inject bad code into `main`.
{:.figcaption}

Looks like `faux-colin` will have to submit a PR. He can't just sneak bad code into the codebase - at least, not into the `main` branch, without a code review.

## Attempt to Deploy a Bad Branch to Prod

`faux-colin` then tries his second attack vector: he'll inject bad code into a branch and then deploy _that_ to production! He commits bad code to an innoucously named branch: `faux-colin-patch-2` for example. No PR - wouldn't want anyone blocking the bad code, now would we? Now to sneak that code into production, he goes to the Actions tab and queues the Deployment Pipeline, selecting his malicious branch as the source:

![Queuing a deployment containing malicious code](/assets/images/2021/11/compliance-reusable/queue-bad-code.png){: .center-image }

Queuing a deployment containing malicious code.
{:.figcaption}

Ha! Bad code on its way...

Except that the branch protection policy kicks in and prevents the deployment to prod!

![Branch protection policy rejecting prod deployment](/assets/images/2021/11/compliance-reusable/no-prod-deploy.png){: .center-image }

Branch protection policy rejecting prod deployment.
{:.figcaption}

At worst, the malicious code is now in the dev environment. You could technically add approvals to the dev environment too, but if you've walled off your dev/prod environments correctly, the risk of malicious code in dev should be minimal. You have to trust your developers at some stage of the process, otherwise people will be totally bogged down in red tape. At least you can rest assured that prod environments are still protected.

## Attempt to Change Deployment Steps

`faux-colin` then decides to change the workflows. Perhaps he doesn't want code scanning to uncover the vulnerability he's introducing, so he'll just bypass the code scanning workflow. So he opens up the `.github/workflows/compliant-scan.yml` and removes the call to the reusable workflow:

~~~yml
{% raw %}
# file: 'APP-REPO/.github/workflows/compliant-scan.yml'
name: Scan app

on:
  workflow_dispatch:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]

jobs:
  scan:
    #name: Code scan
    #uses: colinsalmcorner/super-approved-workflows/.github/workflows/codeql-scan.yml@main
    #with:
    #  languages: '["csharp"]'
    runs-on: ubuntu-latest
    steps:
    - run: echo Code is secure
{% endraw %}
~~~

Attempting to modify the `compliant-scan.yml` file.
{:.figcaption}

Once again, he can't do this on `main` so he has to create a branch and a PR. Interestingly, GitHub is smart enought to know that even though the workflow executed on the PR branch is the required workflow, it still required a check from the workflow in the `main` branch. In the screenshot below, you can see that the "bad" workflow (`Scan app / scan (pull_request)`) is passing, but the check is still blocked because the "good" workflow (`Code scan / Analyze (csharp)`) hasn't been run:

![Unable to bypass the Code Scan workflow requirement](/assets/images/2021/11/compliance-reusable/bypass-code-scan-fail.png){: .center-image }

Unable to bypass the Code Scan workflow requirement.
{:.figcaption}

## Attempt to Change `CODEOWNERS`

`faux-colin` then decides to see if he can jimmy the `CODEOWNERS` file. He tries to add himself as a code owner:

~~~yml
{% raw %}
# Changes to `compliant-` workflows requires @compliance-team approval
/.github/workflows/compliant-* @colinsalmcorner/compliance-team @faux-colin

# Changes to `CODEOWNERS` requires @compliance-team approval
/.github/CODEOWNERS @colinsalmcorner/compliance-team @faux-colin
{% endraw %}
~~~

Attempt to modify the `CODEOWNERS` file to add `faux-colin` as a code owner for the workflows.
{:.figcaption}

Once again our hacker is foiled! A PR now contains his attempted modifications and merging would require approvals from the Compliance Team:

![PR blocks unapproved changes to the CODEOWNERS file](/assets/images/2021/11/compliance-reusable/attempted-codeowner-change.png){: .center-image }

PR blocks unapproved changes to the `CODEOWNERS` file.
{:.figcaption}

# Caring About Culture

It seems that `faux-colin` has not been able to inject malicious code into the application. Of course, this gives the Compliance Team peace of mind: after all, malicious developers are probably few and far between. However, if a malicious developer can't bypass the process, then there's little chance that a developer will _mistakenly_ do something bad. There are enough checks and balances in the configuration.

That means that the Compliance Team have done their job and can _get out of the way_ and let the App Team do what they do best - code, and hopefully innovate! Remember, DevOps (or DevSecOps if you really prefer) is _cultural_ too. Here we have a good balance of process adherence and compliance without developers being overburdened with red tape or the Compliance Team becoming a bottleneck because they have to enforce draconian policies manually.

Caring about the culture your team works under is critical to success today. After all, Talent Management is one of the four top [Developer Velocity](https://www.mckinsey.com/industries/technology-media-and-telecommunications/our-insights/developer-velocity-how-software-excellence-fuels-business-performance) _business_ performance indicators. If you can ensure your code is safe and compliant _and_ foster a positive culture, you're winning. And using GitHub, you can!

# Conclusion
Using a combination of branch protection policies, permission management, reusable workflows, environment approvals and `CODEOWNERS` file, teams can achieve a good combination of autonomy and enterprise alignment. Compliance Teams can rest assured that the process is being enforced without becoming blockers. Developers are happy, Compliance is happy - and business will boom.

Happy complying!
