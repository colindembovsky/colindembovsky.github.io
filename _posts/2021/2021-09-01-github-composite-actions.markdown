---
layout: post
title: GitHub Composite Actions
featured: true
date: '2021-09-01 17:22:01'
image: /assets/images/posts/actions.png
description: >
  Composite Actions now allow you to run other Actions, not just script steps. This is great for composability and maintainability, but there are some limitations that you should be aware of.
tags:
- actions
- build
---

1. TOC
{:toc}

> Resistance is futile.  
> _The Borg. And GitHub._

<!--kg-card-end: markdown-->

> Edit: Thanks to [Tiago](https://twitter.com/tspascoal) for pointing out that you can have more than one Action in a repo.

GitHub Actions has rapidly become one of the most widely used CI/CD system on the planet. However, despite massive adoption, it is still fairly immature as a product, certainly when compared to [Azure Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/). That will inevitably change as the Product Team iterates - but at the moment, there are some limitations that make Actions hard to use in many enterprise scenarios.

The biggest drawback to date has been the fact that there is very limited support for _composability_ in Actions. That is, there are not templates that can be reused.

> To those who are paying attention, Actions does have "templates" but these are more like _starter pipelines_ - they are not composable or callable - they simply give a suggested starting point based on the language found within a repo.

[Composite Actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action) have been around for a while - however, they were limited to running scripts. While this may allow some reusability, not being able to run other Actions was a severe limitation. Fortunately, you can now [run other Actions](https://github.blog/changelog/2020-08-07-github-actions-composite-run-steps/) from within a Composite Action.

While this is certainly a step in the right direction, there are still some limitations and gotchas that I want to explore in this post.

### Sidetrail: Azure Pipeline Templates

Before we continue analyzing Composite Actions, I think it's worthwhile considering templating in Azure Pipelines. Pipelines has a very strong templating system. There are several types of templates:

- Variable templates - for templatizing common variables
- Step templates - for templatizing common steps
- Job and Stage templates - for templatizing entire jobs and stages

These templates are fully featured - that is, steps in a template have exactly the same operation and limitations as steps inline in a Pipeline. As we will see, this is not true for Composite Actions, which is why I mention it here.

### Composite Actions

If I compare Azure Pipeline templates and Composite Actions, I would liken Composite Actions to Step templates in Azure Pipelines: that is, they really serve to group and parameterize sets of steps.

Below we'll look at an example. Before we do, let's consider the limitations that Composite Actions have:

- You cannot pass in complex objects (like arrays of steps)
- ~~You cannot use `if` conditions for steps~~ **Update 10/11/2021** `if` is [now supported](https://github.blog/changelog/2021-11-09-github-actions-conditional-execution-of-steps-in-actions/) in Composite Actions
- Composite Actions cannot read `secrets` - you have to pass secrets in as parameters
- The Actions log does not show a separate log per step as you would see in a "normal" Action - all the steps of the Composite are executed as if they were a single step, making debugging Composite Action logs harder to analyze:
<figure class="kg-card kg-image-card"><img src="/assets/images/2021/9/1175_image.png" class="kg-image" alt loading="lazy"></figure>

> Note how the entire Composite Action only shows as a single step in the log, even though there are multiple steps in the Composite itself.

Even though there are some limitations, they are certainly a vast improvement to Actions. Hopefully we'll see more features evolve in this space to allow better composition and sharing of Actions.

## Case Study: eShopOnContainers

A few months ago I got to do some work on the documentation for [DevOps for ASP.NET Core Developers](https://docs.microsoft.com/en-us/dotnet/architecture/devops-for-aspnet-developers/). The repo with example is on [GitHub](https://github.com/dotnet-architecture/eShopOnContainers). The sample application runs as a set of microservices on Kubernetes. Let's take a look at the Action to build the `basketAPI`:

~~~yaml
{% raw %}
    name: basket-api
    
    on:
      workflow_dispatch:
      push:
        branches:
        - dev
    
        paths:
        - src/BuildingBlocks/**
        - src/Services/Basket/**
        - .github/workflows/basket-api.yml
      
      pull_request:
        branches:
        - dev
    
        paths:
        - src/BuildingBlocks/**
        - src/Services/Basket/**
        - .github/workflows/basket-api.yml
    env:
      SERVICE: basket-api
      IMAGE: basket.api
      DOTNET_VERSION: 5.0.x
    
    jobs:
    
      BuildContainersForPR_Linux:
        runs-on: ubuntu-latest
        if: ${{ github.event_name == 'pull_request' }}
        steps:
        - name: 'Checkout Github Action'
          uses: actions/checkout@master
        
        - name: Setup dotnet
          uses: actions/setup-dotnet@v1
          with:
            dotnet-version: ${{ env.DOTNET_VERSION }}
    
        - name: Build and run unit tests
          run: |
            cd src
            dotnet restore "eShopOnContainers-ServicesAndWebApps.sln"
            cd Services/Basket/Basket.API
            dotnet build --no-restore
            cd -
            cd Services/Basket/Basket.UnitTests
            dotnet build --no-restore
            dotnet test --no-build -v=normal
    
        - name: Compose build ${{ env.SERVICE }}
          run: sudo -E docker-compose build ${{ env.SERVICE }}
          working-directory: ./src
          shell: bash
          env:
            TAG: ${{ env.BRANCH }}
            REGISTRY: ${{ secrets.REGISTRY_ENDPOINT }}
    
      BuildLinux:
        runs-on: ubuntu-latest
        if: ${{ github.event_name != 'pull_request' }}
        steps:
        - name: 'Checkout Github Action'
          uses: actions/checkout@master
    
        - name: Setup dotnet
          uses: actions/setup-dotnet@v1
          with:
            dotnet-version: ${{ env.DOTNET_VERSION }}
    
        - name: Build and run unit tests
          run: |
            cd src
            dotnet restore "eShopOnContainers-ServicesAndWebApps.sln"
            cd Services/Basket/Basket.API
            dotnet build --no-restore
            cd -
            cd Services/Basket/Basket.UnitTests
            dotnet build --no-restore
            dotnet test --no-build -v=normal
    
        - name: Enable experimental features for the Docker daemon and CLI
          run: |
              echo $'{\n "experimental": true\n}' | sudo tee /etc/docker/daemon.json
              mkdir -p ~/.docker
              echo $'{\n "experimental": "enabled"\n}' | sudo tee ~/.docker/config.json
              sudo service docker restart
              docker version -f '{{.Client.Experimental}}'
              docker version -f '{{.Server.Experimental}}'
    
        - name: Login to Container Registry
          uses: docker/login-action@v1
          with:
            registry: ${{ secrets.REGISTRY_HOST }}
            username: ${{ secrets.USERNAME }}
            password: ${{ secrets.PASSWORD }}
    
        - name: Set branch name as env variable
          run: |
            currentbranch=$(echo ${GITHUB_REF##*/})
            echo "running on $currentbranch"
            echo "BRANCH=$currentbranch" >> $GITHUB_ENV
          shell: bash
    
        - name: Compose build ${{ env.SERVICE }}
          run: sudo -E docker-compose build ${{ env.SERVICE }}
          working-directory: ./src
          shell: bash
          env:
            TAG: ${{ env.BRANCH }}
            REGISTRY: ${{ secrets.REGISTRY_ENDPOINT }}
    
        - name: Compose push ${{ env.SERVICE }}
          run: sudo -E docker-compose push ${{ env.SERVICE }}
          working-directory: ./src
          shell: bash
          env:
            TAG: ${{ env.BRANCH }}
            REGISTRY: ${{ secrets.REGISTRY_ENDPOINT }}
    
        - name: Create multiarch manifest
          run: |
            docker --config ~/.docker manifest create ${{ secrets.REGISTRY_ENDPOINT }}/${{ env.IMAGE }}:${{ env.BRANCH }} ${{ secrets.REGISTRY_ENDPOINT }}/${{ env.IMAGE }}:linux-${{ env.BRANCH }}
            docker --config ~/.docker manifest push ${{ secrets.REGISTRY_ENDPOINT }}/${{ env.IMAGE }}:${{ env.BRANCH }}
          shell: bash
{% endraw %}
~~~

This file is 126 lines long - which is not too bad for a single service. But there are 14 services! Before Composite Actions, you had no option but to copy/paste the code for each microservice. And that's bad - since copy/paste inevitably leads to errors. And even if you don't fat-finger it, what if you need to change something in the build process? Now you have to update 14 files. There are also deployment Actions for all the services - so now we have 28 files to maintain!

When we analyze the common steps, there are 4 logical actions:

1. Build an image
2. Run tests, then build an image
3. Build and push an image
4. Deploy a helm chart

The steps for these logical actions are the same if we can parameterize them appropriately.

Have a look at the `BuildLinux` job above - this is the job that executes steps to "Build and push an image". Here is what this job looks like if we refactor the steps into a Composite Action (which we'll see next):

~~~yaml
{% raw %}
      BuildLinux:
        runs-on: ubuntu-latest
        if: ${{ github.event_name != 'pull_request' }}
        steps:
        - name: Checkout code
          uses: actions/checkout@v2
        - uses: ./.github/workflows/composite/build-push
          with:
            service: ${{ env.SERVICE }}
            registry_host: ${{ secrets.REGISTRY_HOST }}
            registry_endpoint: ${{ secrets.REGISTRY_ENDPOINT }}
            image_name: ${{ env.IMAGE }}
            registry_username: ${{ secrets.USERNAME }}
            registry_password: ${{ secrets.PASSWORD }}
{% endraw %}
~~~

That's much better! We're checking out the repo using `actions/checkout@v2` (we need to do this to get access to the code for the Composite Actions as well as the application code) and then we're executing a Composite Action that is going to run all the steps we had inline previously.

> Note how we reference an Action in the local repo, using the full path to the folder (not the yaml file).

If Composite Actions could read `secrets`, we'd save another 4 lines. However, since `secrets` are not readable within Composite Actions, we have to pass them in as parameters.

Let's have a look at the `action.yml` for this Composite Action:

~~~yaml
{% raw %}
    name: "Build and push image"
    description: "Builds and pushes an image to a registry"
    
    inputs:
      service:
        description: "Service to build"
        required: true
      registry_host:
        description: "Image registry host e.g. myacr.azureacr.io"
        required: true
      registry_endpoint:
        description: "Image registry repo e.g. myacr.azureacr.io/eshop"
        required: true
      image_name:
        description: "Name of image"
        required: true
      registry_username:
        description: "Registry username"
        required: true
      registry_password:
        description: "Registry password"
        required: true
      
    runs:
      using: "composite"
      steps:
      - name: Enable experimental features for the Docker daemon and CLI
        shell: bash
        run: |
            echo $'{\n "experimental": true\n}' | sudo tee /etc/docker/daemon.json
            mkdir -p ~/.docker
            echo $'{\n "experimental": "enabled"\n}' | sudo tee ~/.docker/config.json
            sudo service docker restart
            docker version -f '{{.Client.Experimental}}'
            docker version -f '{{.Server.Experimental}}'
    
      - name: Login to Container Registry
        uses: docker/login-action@v1
        with:
          registry: ${{ inputs.registry_host }}
          username: ${{ inputs.registry_username }}
          password: ${{ inputs.registry_password }}
    
      - name: Set branch name as env variable
        run: |
          currentbranch=$(echo ${GITHUB_REF##*/})
          echo "running on $currentbranch"
          echo "BRANCH=$currentbranch" >> $GITHUB_ENV
        shell: bash
    
      - name: Compose build ${{ inputs.service }}
        shell: bash
        run: sudo -E docker-compose build ${{ inputs.service }}
        working-directory: ./src
        env:
          TAG: ${{ env.BRANCH }}
          REGISTRY: ${{ inputs.registry_endpoint }}
    
      - name: Compose push ${{ inputs.service }}
        shell: bash
        run: sudo -E docker-compose push ${{ inputs.service }}
        working-directory: ./src
        env:
          TAG: ${{ env.BRANCH }}
          REGISTRY: ${{ inputs.registry_endpoint }}
    
      - name: Create multiarch manifest
        shell: bash
        run: |
          docker --config ~/.docker manifest create ${{ inputs.registry_endpoint }}/${{ inputs.image_name }}:${{ env.BRANCH }} ${{ inputs.registry_endpoint }}/${{ inputs.image_name }}:linux-${{ env.BRANCH }}
          docker --config ~/.docker manifest push ${{ inputs.registry_endpoint }}/${{ inputs.image_name }}:${{ env.BRANCH }}
{% endraw %}
~~~

Nothing too complex here - we have `name` and `description` attributes, followed by a map of input parameters. Each parameter has a `description` and `required` attribute, and can optionally have a `default` attribute too. Then we have a `runs` keyword with the `using` set to `composite`. Thereafter, we have steps as we would in any other Action - including the ability to use other Actions (not only `run` scripts)!

> Note that in Composite Actions, each `run` step requires that you explicitly define the `shell`.

## Conclusion

Composite Actions are a welcome addition to the Actions ecosystem, despite their limitations. I highly recommend that you start using them to reduce copy/paste and keep you and your Actions DRY!

Happy building!

