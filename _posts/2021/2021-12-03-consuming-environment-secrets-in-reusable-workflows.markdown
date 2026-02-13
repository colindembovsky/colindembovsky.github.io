---
layout: post
title: 'Consuming Environment Secrets in Reusable Workflows'
date: '2021-12-03 01:22:01'
image: /assets/images/2021/12/wf-env-sec/key3.jpg
description: >
  One canonical use of reusable workflows is a reusable deployment job. While this is definitely possible with reusable workflows, it's not easy to get it working. In this post I'll show you how to do it.
tags:
- build
- deployment
---

1. TOC
{:toc}

> Image from user15245033 on [www.freepik.com](https://www.freepik.com/vectors/gold)

Last night I was about to go to bed when [Damien Brady](https://github.com/damovisa), [Chris Patterson](https://github.com/chrispat) and I got into a Slack discussion about how to use environments, and specifically secrets, with reusable workflows.

The [documentation ](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows#using-inputs-and-secrets-in-a-reusable-workflow) explains that reusable workflows can access secrets via the `secrets` keyword, and does mention environments, but it's not very clear about how to get environment secrets into a reusable workflow.

# tl;dr

If you just want the summary about how to use environment secrets in reusable workflows, then just follow these steps:

{% raw %}
1. Define secrets in your environment
1. Make `environment` an input parameter to your reusable workflow
1. Use `environment: ${{ inputs.environment }}` inside the `job` within the reusable workflow
1. Declare the `secrets` your reusable workflow requires alongside the `inputs`
1. When you call the reusable workflow, pass in the secrets
{% endraw %}

A repo with the workflows is [here](https://github.com/colindembovsky/reusable-workflows-env-secrets). For a longer explanation and some examples, keep on reading!

# Simple Steps

To keep things simple, let's consider this step as the "heart" of the reusable workflow:

~~~yml
{% raw %}
steps:
- name: Dump Password
  run: |
    echo Password is $PASSWORD
    if [[ $PASSWORD == *"PROD"* ]]; then
      echo "This is the PROD password!"
    else
      echo "This is NOT the PROD password!"
    fi
  env:
    PASSWORD: ${{ secrets.PASSWORD }}
{% endraw %}
~~~

Heart of the reusable workflow.
{:.figcaption}

Of course, in real life you'd never output any information about a password, but since secrets are masked to look like `***` in the logs, I wanted some way to confirm that we're getting the correct passwords for the corresponding environment.

## Environments

Next, let's create two environments: `dev` and `prod`. I'll navigate to **Settings->Environments** and create the environments. Then I create a secret called `PASSWORD` in each environment, with the value `DEV_PASSWORD` for `dev` and `PROD_PASSWORD` for `prod`.

![Setting up a dev secret](/assets/images/2021/12/wf-env-sec/dev-password-secret.png){: .center-image }

Setting up a dev secret.
{:.figcaption}

With that setup out of the way, we can attempt to use secrets in a reusable workflow!

# Attempt 1: Specify Environment and Uses Together

Let's try this for the reusable workflow:

~~~yml
{% raw %}
# file: '.github/workflows/reusable-deploy.yml'
name: Deploy Template

on:
  workflow_call:

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    
    steps:
    - name: Dump Password
      run: |
        echo Password is $PASSWORD
        if [[ $PASSWORD == *"PROD"* ]]; then
          echo "This is the PROD password!"
        else
          echo "This is NOT the PROD password!"
        fi
      env:
        PASSWORD: ${{ secrets.PASSWORD }}
{% endraw %}
~~~

Attempt 1 reusable workflow.
{:.figcaption}

Seems reasonable. We're executing a job and (we hope) it's in the context of an environment, so we can access the secrets defined on the environment.

Here's our attempt at the caller workflow:

~~~yml
{% raw %}
# file: '.github/workflows/deploy-pipeline.yml'
name: Deploy Pipeline

on:
  workflow_dispatch:

jobs:
  deploy_dev:
    name: Deploy to Dev
    environment: dev
    uses: colindembovsky/reusable-workflows-env-secrets/.github/workflows/reusable-deploy.yml@main
      
  deploy_prod:
    name: Deploy to Prod
    needs: deploy_dev
    environment: prod
    uses: colindembovsky/reusable-workflows-env-secrets/.github/workflows/reusable-deploy.yml@main
{% endraw %}
~~~

Attempt 1 caller workflow.
{:.figcaption}

Again, this seems reasonable. We're reusing the `reusable-deploy.yml` workflow file, and we're telling Actions which environment we're running in. Everything from there should just be wired up, right?

Not exactly. If you're editing the workflow, you'll see some helpful red squiggles:

![You can't use environment with uses](/assets/images/2021/12/wf-env-sec/cant-mix-uses-env.png){: .center-image }

You can't use `environment` with `uses`.
{:.figcaption}

That's not cool.

At this point we could refactor the reusable workflow into a [composite workflow]({% post_url 2021-09-01-github-composite-actions %}) so that we can use the `environment`. Or...

# Attempt 2: Make Environment an Input

Ok Actions, we're not able to make it work "the easy way". We'll just pass the environment into the reusable workflow - then we'll have access to the secrets!

Let's try this for our new reusable workflow:

~~~yml
{% raw %}
# file: '.github/workflows/reusable-deploy.yml'
name: Deploy Template

on:
  workflow_call:
    inputs:
      environment:
        type: string
        description: environment to deploy to
        required: true

jobs:
  deploy:
    name: Deploy to ${{ inputs.environment }}
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    
    steps:
    - name: Dump Password
      run: |
        echo Password is $PASSWORD
        if [[ $PASSWORD == *"PROD"* ]]; then
          echo "This is the PROD password!"
        else
          echo "This is NOT the PROD password!"
        fi
      env:
        PASSWORD: ${{ secrets.PASSWORD }}
{% endraw %}
~~~

Attempt 2 reusable workflow, with `environment` as an input.
{:.figcaption}

And for the caller workflow, we'll attempt this:

~~~yml
{% raw %}
# file: '.github/workflows/deploy-pipeline.yml'
name: Deploy Pipeline

on:
  workflow_dispatch:

jobs:
  deploy_dev:
    name: Deploy to Dev
    uses: colindembovsky/reusable-workflows-env-secrets/.github/workflows/reusable-deploy.yml@main
    with:
      environment: dev
      
  deploy_prod:
    name: Deploy to Prod
    needs: deploy_dev
    uses: colindembovsky/reusable-workflows-env-secrets/.github/workflows/reusable-deploy.yml@main
    with:
      environment: prod
{% endraw %}
~~~

Attempt 2 caller workflow, passing `environment` as an input.
{:.figcaption}

No red squiggles! Yay!

Unfortunately, when we run the workflow, the `PASSWORD` secret is empty:

![The PASSWORD secret is empty](/assets/images/2021/12/wf-env-sec/empty-password.png){: .center-image }

The `PASSWORD` secret is empty!
{:.figcaption}

We're not stuck yet - let's press on and try to specify the secrets we need.

# Attempt 3: Pass the Environment and Secrets

The documentation hints that we should be able to do this. We're close. Let's make one more attempt:

~~~yml
{% raw %}
# file: '.github/workflows/reusable-deploy.yml'
name: Deploy Template

on:
  workflow_call:
    inputs:
      environment:
        type: string
        description: environment to deploy to
        required: true
    secrets:
      PASSWORD:
        required: true

jobs:
  deploy:
    name: Deploy to ${{ inputs.environment }}
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    
    steps:
    - name: Dump Password
      run: |
        echo Password is $PASSWORD
        if [[ $PASSWORD == *"PROD"* ]]; then
          echo "This is the PROD password!"
        else
          echo "This is NOT the PROD password!"
        fi
      env:
        PASSWORD: ${{ secrets.PASSWORD }}
{% endraw %}
~~~

Attempt 3 reusable workflow, with `environment` as an input and a `secrets` section.
{:.figcaption}

All we're doing here is explicitly defining which secrets our reusable workflow needs.

For the caller workflow, we'll attempt this:

~~~yml
{% raw %}
# file: '.github/workflows/deploy-pipeline.yml'
name: Deploy Pipeline

on:
  workflow_dispatch:

jobs:
  deploy_dev:
    name: Deploy to Dev
    uses: colindembovsky/reusable-workflows-env-secrets/.github/workflows/reusable-deploy.yml@main
    with:
      environment: dev
    secrets:
      PASSWORD: ${{ secrets.PASSWORD }}
      
  deploy_prod:
    name: Deploy to Prod
    needs: deploy_dev
    uses: colindembovsky/reusable-workflows-env-secrets/.github/workflows/reusable-deploy.yml@main
    with:
      environment: prod
    secrets:
      PASSWORD: ${{ secrets.PASSWORD }}
{% endraw %}
~~~

Attempt 3 caller workflow, passing `environment` as an input as well as the `secrets`.
{:.figcaption}

It works!

![The PASSWORD in dev](/assets/images/2021/12/wf-env-sec/dev-working.png){: .center-image }

The `PASSWORD` is correct in dev.
{:.figcaption}

![The PASSWORD in prod](/assets/images/2021/12/wf-env-sec/dev-working.png){: .center-image }

The `PASSWORD` is correct in prod.
{:.figcaption}

# What Gives?

What is happening here?

This is now working because we're explicitly defining _which_ secrets from the environment the reusable workflow is able to read. We do this by defining the `secret` in the `workflow_call` of the reusable workflow, as well as passing in the secret from the caller workflow.

The product team made this decision so that you would always know explicitly which secrets a reusable workflow has access to. It makes reusable workflows more secure, but makes understanding the flow and authoring a little trickier.

Another confusing notion is the meaning of the secret from the caller perspective:

~~~yml
{% raw %}
jobs:
  deploy_dev:
    name: Deploy to Dev
    uses: colindembovsky/reusable-workflows-env-secrets/.github/workflows/reusable-deploy.yml@main
    with:
      environment: dev
    secrets:
      PASSWORD: ${{ secrets.PASSWORD }}
{% endraw %}
~~~

What does `secrets.PASSWORD` mean here?
{:.figcaption}

{% raw %}
If you look closely at the caller workflow, you'll see we're passing `${{ secrets.PASSWORD }}` to the `PASSWORD` secret of the reusable workflow. In this context, the value is actually meaningless, since we're not really "inside" and environment (the workflow doesn't know what the `environment` value means - it's just a string value). What we're really doing is passing the _key_ to the secret that we want the reusable workflow to use.
{% endraw %}

# Limitations

If you've got a small number of secrets (say less than 10), then this technique works. If you've got more secrets for your environments, then you should probably look at storing your secrets in a secret store (like Azure KeyVault or HashiVault) and add steps in your workflow to retrieve the secrets you need from the vault. Then the only secret you really need is the credential to the vault!

# Conclusion
Reusable workflows are great - but understanding how environment secrets work with reusable workflows can be challenging. Hopefully this post gives you some insight into how things work so that you can start authoring reusable deployment jobs that use environment secrets!

Happy reusing!