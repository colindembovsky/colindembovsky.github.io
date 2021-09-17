---
layout: post
title: 'Deployment with GitHub Actions: The Bad and the Ugly'
date: '2021-01-11 22:42:40'
description: >
  GitHub Actions can be used for Continuous Deployment (CD) - but there are some rough edges. In this post I take you through a deep dive and lift the kimono on Actions.
tags:
- actions
---

1. TOC
{:toc}

Let me start this post with three disclaimers:

1. [GitHub Actions Environments](https://docs.github.com/en/free-pro-team@latest/actions/reference/environments) is in beta
2. I'm a huge fan of [Azure Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/), which is a fairly mature product
3. The GitHub engineers (and PM [Chris Patterson](https://github.com/chrispat)) have been amazing with answering my questions and helping me deep dive into Actions for CD

Why start with disclaimers? Because (SPOILER ALERT) deployment with GitHub Actions is rough. Unfortunately, not even enough "good" to add "The Good" to the title of this post.

> Note: for ease of typing and reading, I'll refer to GitHub Actions as simply Actions, and Azure Pipelines as simply Pipelines for the remainder of this post.

Ever since Actions was born, the vision has been "code to cloud". This vision is exactly correct and certainly demos nicely if you have a simple web app and a single environment. But how do you handle a more complex real-world scenario? How do you deploy and configure infrastructure? How do you model multiple environments? Where do you store secrets and environment settings? How do you handle approvals?

At GitHub Universe in December 2020, GitHub released Environments to Beta. This was the first baby step in moving towards a more "enterprise" deployment capability.

## Environments, Stages, Jobs and Templates

I've made a few Actions workflows and have always approached it as a slightly differently formatted Azure Pipeline that can only have jobs (no stages or environments). So when Actions Environments got released, I thought I'd approach Actions with the same paradigm I use when I create Pipelines.

I was wrong.

Firstly, I was annoyed that there is no such thing as a "[stage](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/stages?view=azure-devops&tabs=yaml)" in Actions. When writing Pipelines, I usually equate the _environment_ with a _stage_ and then run multiple jobs in each stage. This lets me create job [templates](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops) that I can reuse. Deployments should be as similar as possible between environments which I can easily model with parameterized templates.

But Actions doesn't have templates - though it's on the [roadmap](https://github.com/github/roadmap/issues/98). For now, I'll just have to live with some copy-and-paste.

While there are no stages, I could conceptually map jobs to stages in an Action workflow, and use the `environment` attribute to tell GitHub that a job is targeting an environment. If I need multiple jobs for a stage, I can use a naming convention to loosely group them, even if the UI won't.

Except that Actions behaves strangely when you have multiple jobs targeting a single environment (more about this later).

## Infrastructure and Code

I bounce back and forth between having infrastructure and app code in the same pipeline or having separate pipelines for these concerns. Using Pipelines, I typically have an infrastructure job to get the environment resources spun up and/or configured, and then have a deployment job to deploy app code, and then have one or more test jobs for running tests. I always use a single build job to produce a single binary that is promoted between environments.

However, this can complicate approvals. Typically, ops folks will approve infrastructure changes, while dev folks will approve code deployments. As much as we should be working on cross-functional teams, it's all too common to still see silos in the enterprise. And even on cross-functional teams, we're usually all T-shaped - that is, we're usually more "ops-ey" or more "dev-ey" and so approvals tend to fall along that divide. This scenario works fine with Pipelines since each job in the stage is targeting an environment and the approvals are defined on the environment. There's no way of specifying who exactly should do the review, so the team will have to work out who's approval is currently pending (is it for infra or for code). GitHub Environments have the same limitation.

After chatting through some of the challenges with Chris Patterson (PM for Actions) he suggested splitting infra and code into separate pipelines. I can at least trigger the infra pipeline only if infra code changes, and trigger the app pipeline for everything else this way.

## TailWind Web App

Let's make this a bit more concrete. For ease of use, I'll be using a DotNetCore app called TailWind originally from Microsoft, just so that I have some code I can deploy to Azure. I forked the [Microsoft repo](https://github.com/microsoft/TailwindTraders-Website) and added in Terraform templates for the infra into [my version](https://github.com/10thmagnitude/TailwindTraders-Website). Now I have infra-as-code as well as the app code.

Using Azure Pipelines, I would have a workflow that looks something like this:

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/images/2021/1/111917_image.png" class="kg-image" alt loading="lazy"><figcaption>Azure Pipeline for deploying TailWind Web App</figcaption></figure>

There are multiple stages/jobs here:

<!--kg-card-begin: markdown-->
1. Stage 1: Build and test sources
  - this builds and packages the code and runs unit tests
2. Stage 2: Deploy to DEV
  - Run terraform to create/configure infrastructure in the DEV environment
  - Deploy the web app using the package from Stage 1
  - Run Selenium (coded UI) tests
3. Stage 3: Canary deployment to PROD
  - Run terraform to create/configure infrastructure in the PROD environment
  - Deploy the web app using the package from Stage 1
  - Route traffic to the staging slot
  - Run JMeter (load) tests
4. Stage 4: Promote or Rollback
  - Depending on the outcome of the tests against the Canary slot, we can either promote to PROD (slot swap) or simply route all traffic back to the production slot (this is effectively a rollback)
<!--kg-card-end: markdown-->

All the jobs in the Pipeline are templatized - you'll note that I use the same `Run terraform` and `Deploy the web app` jobs in stage 2 and 3, just passing in environment specific values.

Since I don't have stages in Actions, my initial Action workflow looked like this:

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/images/2021/1/111933_image.png" class="kg-image" alt loading="lazy"><figcaption>End to end Action</figcaption></figure>

Here I have the following jobs:

1. build - builds and packages code (and would run unit tests here if I had any)
2. dev\_infra\_plan - runs `terraform plan` against the DEV environment
3. dev\_infra\_apply - downloads the output plan from the previous job and runs `terraform apply`
4. dev\_deploy - deploys web app package from job 1 to the DEV environment
5. prod\_infra\_plan - runs `terraform plan` against the PROD environment
6. prod\_infra\_apply - downloads the output plan from the previous job and runs `terraform apply`
7. prod\_deploy - deploys web app package from job 1 to the PROD environment

Yes, I had to use copy and paste for jobs 2 and 5, 3 and 6 and 4 and 7. But I know templates are coming, so I could live with that for a while. However, the job got stuck in this state and I could never get to job 5 (the start of the PROD jobs). That's when Chris Patterson helped me understand that because I have multiple jobs targeting the same environment(s) strange things are happening...

So it seems you shouldn't have _multiple jobs targeting the same environment_. And you can't have templates. Now I was faced with giant blocks of Actions that were reduced into BUILD-\>DEV-\>PROD with a single job for each. And a single approval for each.

Horrible. Just horrible.

## Interlude: Terraform Wrapper

Before I get to some workarounds for some of the challenges, I want to mention how I lost several hours of my life. Take a look at this snippet:

~~~yaml
{% raw %}
    - name: Install TF
      uses: hashicorp/setup-terraform@v1
      with:
        terraform_version: ${{ env.tf_version }}
    
    - name: Init TF
      run: terraform init --backend-config="key=${{ env.env_name }}.terraform.tfstate"
    
    - name: TF Output web app URL
      run: |
        url=$(terraform output -raw webAppURL)
        echo "The URL is $url"
{% endraw %}
~~~

This workflow snippet is really modelling the following commands:

~~~yaml
{% raw %}
    terraform init --backend-config="key=DEV.terraform.tfstate"
    url=$(terraform output -raw webAppURL)
    echo "The URL is $url"
{% endraw %}
~~~

When running these locally, I get `The URL is http://<url>` which is what I expect.

When running the workflow, I could not get this to work. I wasted a lot of time trying to figure out what I was doing wrong. The message looked something like this:

```
The url is [command]/home/runner/work/_temp/056bc345-efd0-4b6d-ae6c-94094f124a7f/terraform-bin output -raw
```

Turns out that all I had done wrong was make an assumption about how the `hashicorp/setup-terraform@v1` task works. I assumed it just installed `terraform`. However, in this configuration, it installs `terraform` and creates a wrapper around the executable, so you cannot treat terraform commands as you would locally - you have to access the output parameters.

To be fair, the Hashi folks make this clear on their [documentation](https://github.com/hashicorp/setup-terraform), but I didn't think I'd need to read the docs for running an install! I logged [this issue](https://github.com/hashicorp/setup-terraform/issues/85) for them to update the logging so that others who make the same assumption don't waste hours before realizing that the Action installs a wrapper by default.

So at this point, while I was trying hard to love Actions, it just wasn't looking good.

## The Workaround

> Note: Workflow code is [here](https://github.com/10thmagnitude/TailwindTraders-Website/blob/main/.github/workflows/infra-end-to-end.yaml) for my infra end to end and [here](https://github.com/10thmagnitude/TailwindTraders-Website/blob/main/.github/workflows/app-end-to-end.yaml) for my app end to end.

The first thing to do was to split the infrastructure Actions from the code deployment Actions. This made the workflows more aligned to a single responsibility. Since approvals are still on the environments, I still don't have a good way to separate infra approvals from code deployment approvals. I could create two environment objects in Actions that are conceptually the same environment, but that can get hard to manage if I have secrets (which are defined on the environment). Having two environments with the same settings means I have to duplicate all the settings.

## Secrets and Approvals on Environments

I need to reference DEV for both the `plan` and `approve` jobs so that I can authenticate to the Azure subscription. Ideally, `plan` would run without requiring an approval and then I can review the output of the plan before I run `apply`. However, since I need to reference the environment in both jobs, I have to approve both jobs. Azure Pipelines suffers a similar limitation.

I ended up with three environments: `DEV`, `PROD-PLAN` and `PROD`. I have defined the `AZURE_CREDENTIALS` secret for `PROD` at the repository level. That way, when any job refers to `${{ secrets.AZURE_CREDENTIALS }}` by default it will get the credentials for `PROD`. For jobs running in `DEV`, I explicitly reference the `DEV` environment using the `environment` attribute. That way I can get the credentials for `DEV`.

Why did I default to `PROD` and not to `DEV`? The only reason was to avoid having to approve the `plan` job for `PROD`. What happens now is that I have an approval on the environment `prod` but I don't reference that for the `prod-plan` job. Instead I reference `PROD-PLAN` just so that I know this is a deployment job - but there is no approval on this environment. The value for `${{ secrets.AZURE_CREDENTIALS }}` is the PROD value, so the `terraform plan` goes ahead and performs a plan against `PROD`. The next job, `prod-apply` explicitly references `PROD` and so waits for the approval.

For `DEV` I don't have approvals, so `plan` and `apply`, though both referencing `DEV` so that they get the `DEV` credentials, do not wait for approvals.

Of course, if I had a `UAT` environment, I'd have to choose to split it or just live with having to approve `plan` and `apply` jobs.

In fact, I don't really need the `PROD-PLAN` environment at all in this case - there are no secrets for this faux environment and no approvals. But if infra folks did require an approval before running a `plan` I can just update the environment.

## Manual Triggers

You can create a manual trigger for a workflow - but there are some gotchas. Firstly, you have to specify a `workflow_dispatch` `on` event - even if it's empty like so:

~~~yaml
{% raw %}
    on:
      workflow_dispatch: # manual trigger
      push:
        branches:
        - main
        - actions  
{% endraw %}
~~~

The empty `workflow_dispatch:` looks a bit odd, but that is the correct syntax.

And the "Run Workflow" button **won't appear for that workflow** , even if it has the `workflow_dispatch` trigger, unless that workflow is in the default (`main` or `master` or whatever you've set your default branch to) branch.

## Disappearing History - Sort of

Another issue with the UI is the tree of workflows to the left of the run history UI:

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/images/2021/1/112238_image.png" class="kg-image" alt loading="lazy"><figcaption>Tree listing all workflows in the main branch of the repo</figcaption></figure>

I initially had a workflow called `End to End`. When I split it into two files for `App Deploy End to End` and `Infra Deploy End to End`, the node in the tree for `End to End` disappeared. The runs are still in the list (the 210 results you can see in the screenshot) but I can't filter by that workflow. I'm guessing that the tree on the left is built from _current_ workflows in the .github folder in the main branch - so if you delete or rename a workflow, you'll lose the node on the left.

## Pesky Environment URL

Let's have a look at the `url` attribute of the `environment` in an app deployment workflow job:

~~~yaml
{% raw %}
    prod_deploy:
      needs: prod_deploy_canary
      runs-on: ubuntu-latest
      environment:
        name: PROD
        url: ${{ steps.slot_swap.outputs.url }}
{% endraw %}
~~~

This will set the URL of the job in the UI accordingly so that users can easily link to the environment - as long as it's a _valid_ URL. The URL has to start with `http://` or `https://` otherwise it won't show anything. This foxed me for a while - since some `az cli` commands just output the URL without the protocol, like `somesite.azurewebsites.net`. Remember to prepend `http://` or they won't show.

Another quirk is that the URL updates for all jobs referencing the environment. In my scenario, I have two jobs for the `PROD` environment: a canary deployment that deploys to a staging slot and a "full" deployment that performs a slot swap. For the canary job, the URL is `http://my-site-staging.azurewebsites.net` while for the "full" deployment the URL is `http://my-site.azurewebsites.net`. But both are targeting the `PROD` environment. Here is what the workflow looks like after the canary job:

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/images/2021/1/11214_image.png" class="kg-image" alt loading="lazy"><figcaption>Workflow after the Canary job</figcaption></figure>

However, after the `prod_deploy` job, the URL is updated for `PROD` which both jobs reference, and the UI ends up dropping the canary URL:

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/images/2021/1/11218_image.png" class="kg-image" alt loading="lazy"><figcaption>Canary URL is gone</figcaption></figure>

I'm not sure why Actions does this - perhaps both environments should show the final value? It's strange that the value is wiped.

## Fireside Chat with Chris Patterson

Chris Patterson (PM at Actions) kindly connected with me to walk through some of the challenges I was experiencing and questions I had. We also got to chat about some of the upcoming improvements that he and the team are planning. Rest assured that the GitHub team know that Actions Environments are in beta and have a lot of ideas for future features and improvements. Since Chris was PM of Azure Pipelines, he has a lot of experience with CI/CD tools and has a lot of experience of what worked well and what made life complicated. I don't envy his job: make Actions simple to approach, but powerful. Make it deterministic, but dynamic - it's a very fine balancing act.

## Conclusion

Actions is still relatively young and already it is the most popular CI tool on GitHub. It has some maturing to do to compete in the CD space. For now, I'd still recommend using GitHub for source control and security scanning, and use Azure DevOps for CI/CD Pipelines and Boards.

GitHub Actions still have a ways to go to become a mature CD tool. For now, you're probably better off trying to craft your CD workflows as a single job per environment, splitting infrastructure and code deployments to reduce complexity in the overall workflow. Hopefully we'll get templates too, as well as some way to better manage jobs that need environment values but are not actually deployment jobs (like `terraform plan` which needs values, but does not change the environment). And hopefully being able to have multiple jobs (like canary and full deploy) targeting the same environments will receive a better experience too. Lots of work for the engineering team to do still!

Finally, I'll remind the reader that Actions are in beta - so some rough edges are to be expected. But there is a seed of promise there that we can fully expect to blossom soon!

Happy deploying!

