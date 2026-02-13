---
layout: post
title: Musings on GitHub Actions Reusable Workflows
date: '2021-10-05 01:22:01'
image: /assets/images/2021/10/reuse.jpeg
description: >
  Newly released Reusable Workflows allows you to reuse workflows in your GitHub workflows. While this still has some limitations, it's still better than copy/paste!
tags:
- actions
- build
---

1. TOC
{:toc}

[Reusable workflows](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows) have just been released to beta. Some level of reuse has been possible previously using [Composite Actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action).

I wrote about Composite Actions and some of their limitations [here]({% post_url 2021-09-01-github-composite-actions %}). In that post I also compared Composite Actions to Azure Pipeline `step` templates. Reusable workflows are akin to `job` templates in Azure Pipelines.

## Motivation for Reusable Workflows

Reusable workflows allow you to centralize a set of common jobs. For example, you may have a common job for "Build a .NET application". You may have a common job for "Deploy a Web App to Azure Web Apps". Instead of each repo having the same set of steps for these common jobs, you can now create a centralized repo (or set of repos) that can be reused.

This keeps your workflows DRY (Don't Repeat Yourself) and allows you to change a workflow in a single place. Since you can pin to a tag or branch when _referencing_ a reusable workflow, you can ensure that you maintain backward compatibility.

Many organizations have a "DevOps Team". I really don't like this terminology, since I think DevOps should be everyone's resposibility, not just fobbed off to some other team. However, in practice, many organizations have teams that write application code and have other teams that are focused on automating building, testing, securing and deploying that code. Reusable workflows is great for these teams.

For teams that are building their own automation, they can still benefit from a centralized location for reusable workflows. In this case, anyone could contribute to the workflows, where organizations with "DevOps teams" may want only the DevOps teams to be able to contribute to the workflows. In the end, this becomes a _process_ question rather than a tooling question - you can set permissions (or lack of permissions) on the centralized repos however you like.

## Limitations

There are still some limitations to reusable wofklows:

1. ~~Reusable workflows only run on _hosted_ runners, not yet on _private_ runners.~~ **Update**: Reusable workflows are supported on self-hosted runners now (10/14/2021)!
1. ~~Reusable workflows cannot call other reusable workflows.~~ **Update**: You can nest reusable workflows [up to 4 levels](https://github.blog/changelog/2022-08-22-github-actions-improvements-to-reusable-workflows-2/) now (8/22/2022)!
1. ~~You cannot access job outputs from reusable workflows.~~ **Update**: You can now declare `outputs` to reusable workflows. These work just like `job` outputs and are available via `needs.<reusable>.outputs.<output>` format once you declare the output.
1. `env` variables set in the calling workflow are not accessible to the called workflow.
1. The only parameter types that are supported are `string`, `number` and `boolean`. Arrays are not supported.
1. You cannot pass in steps to inject (like you can with Azure Pipelines templates).
1. Repos that contain reusable workflows must be either `internal` or `public`.
1. You cannot enforce use of a reusable workflow. In other words, there is no equivalent for [extends templates](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops#extend-from-a-template).

These limitations mean that reusable workflows are nowhere near as powerful as Azure Pipeline templates, but it's a step in the right direction for Actions. At the very least, reusable workflows minimized copy/paste for simple scenarios.

## Making a Workflow Reusable

### Add the Call Trigger
Making a workflow reusable is not too hard - you just add a `workflow_call` trigger, with some optional `inputs` and `secrets`. The `workflow_call` trigger allows your workflow to be called by other workflows, and obviously you can pass values for the `inputs` and `secrets` using the `with` keyword, just like any other Action.

### Enable Actions Access on the Repo
The `workflow_call` trigger makes the workflow callable, but you still need to allow access. In the `Settings->Actions` tab of your repo, scroll to the bottom and enable the desired level of access:

![Setting Actions Access level on a repo](/assets/images/2021/10/actions-access.png){: .center-image }

Setting Actions Access level on a repo.
{:.figcaption}

## Creating a Reusable DotNet Build, Test and Publish Workflow

Imagine you have a standard way of building and testing dotnet applications. Before reusable workflows, you would have had to copy/paste a workflow to every dotnet repo. Now you can set up a single workflow that can be reused. Let's take a look at an example:

~~~yml
{% raw %}
# file: 'simple-build.yml'
name: Build dotnet application

on:
  workflow_call:
    inputs:
      dotnet-version:
        description: Version of dotnet to use
        type: string
        default: 5.0.x

jobs:
  build:
    name: Build dotnet app
    
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v2
      - name: Setup .NET
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: ${{ inputs.dotnet-version }}
      - name: Restore dependencies
        run: dotnet restore
      - name: Build
        run: dotnet build --no-restore
      - name: Test
        run: dotnet test --no-build --verbosity normal
{% endraw %}
~~~

A simple workflow to build a dotnet application.
{:.figcaption}

Notes:
1. We specify the `workflow_call` trigger in the `on` section to indicate that this is a reusable workflow.
1. We include an `input` called  `dotnet-version` with some metadata as well as a default value of `5.0.x`
1. The steps are really easy: `checkout` the code, setup the specified version of dotnet and then `run` `restore`, `build` and `test`

So far so good. Let's enhance this workflow to add in some more functionality. 

### Parameterize the Platform

Let's parameterized the `runs-on` so that we can pass in the hosted or private pool we want to run the workflow on:

~~~yml
{% raw %}
on:
  workflow_call:
    inputs:
      ...
      runs-on:
        description: Platform to execute on
        type: string
        default: ubuntu-latest

jobs:
  build:
    name: Build dotnet app
    
    runs-on: ${{ inputs.runs-on }}
    ...
{% endraw %}
~~~

Adding a `runs-on` input.
{:.figcaption}

### Parameterize the Source Directory

What if the project file isn't in the root directory of the repo? Many teams put the app code into a folder called `src` or have a mono-repo with several apps in different folders. Not a problem - we can add a `project-folder` input and set that as the default working directory:

~~~yml
{% raw %}
on:
  workflow_call:
    inputs:
      ...
      project-folder:
        description: The folder containing the project to build
        type: string
        default: .

jobs:
  build:
    name: Build dotnet app
    
    runs-on: ${{ inputs.runs-on }}
    defaults:
      run:
        working-directory: ${{ inputs.project-folder }}
    ...
{% endraw %}
~~~

Adding a `project-folder` input.
{:.figcaption}

### Parameterize Running Tests

What if there are no tests to run or we don't want to run them because they take too long? We can add a boolean parameter and skip the test step if this value is false:

~~~yml
{% raw %}
on:
  workflow_call:
    inputs:
      ...
      run-tests:
        description: Run tests
        type: boolean
        default: true

jobs:
  build:
    name: Build dotnet app
    
    ...
    steps:
    ...
    - name: Test
      if: ${{ inputs.test }}
      run: dotnet test --no-build --verbosity normal
{% endraw %}
~~~

Adding a `run-tests` boolean input.
{:.figcaption}

### Publish the App

What if we want to publish the compiled app, and specify the configuration and name of the published artifact? No problem.

~~~yml
{% raw %}
on:
  workflow_call:
    inputs:
      ...
      publish-configuration:
        description: Configuration to publish
        type: string
        default: Release
        
      artifact-name:
        description: Name of the artifact to upload
        type: string
        default: drop

jobs:
  build:
    name: Build dotnet app
    
    ...
    steps:
    ...
    - name: Publish
      run: dotnet publish -c ${{ inputs.publish-configuration }} -o wdrop
    - name: Upload a Build Artifact
      uses: actions/upload-artifact@v2.2.2
      with:
        name: ${{ inputs.artifact-name }}
        path: ${{ inputs.project-folder }}/wdrop/**
        if-no-files-found: error
{% endraw %}
~~~

Adding steps and inputs for publishing.
{:.figcaption}

### Putting it all together

The final workflow looks like this:

~~~yml
{% raw %}
# file: 'completed-dotnet-build.yml'
name: Build dotnet application

on:
  workflow_call:
    inputs:
      runs-on:
        description: Platform to execute on
        type: string
        default: ubuntu-latest
        
      dotnet-version:
        description: Version of dotnet to use
        type: string
        default: 5.0.x
      
      project-folder:
        description: The folder containing the project to build
        type: string
        default: .
        
      run-tests:
        description: Run tests
        type: boolean
        default: true
      
      publish-configuration:
        description: Configuration to publish
        type: string
        default: Release
        
      artifact-name:
        description: Name of the artifact to upload
        type: string
        default: drop
    
jobs:
  build:
    name: Build dotnet app
    
    runs-on: ${{ inputs.runs-on }}
    defaults:
      run:
        working-directory: ${{ inputs.project-folder }}
    
    steps:
      - uses: actions/checkout@v2
      - name: Setup .NET
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: ${{ inputs.dotnet-version }}
      - name: Restore dependencies
        run: dotnet restore
      - name: Build
        run: dotnet build --no-restore
      - name: Test
        if: ${{ inputs.test }}
        run: dotnet test --no-build --verbosity normal
      - name: Publish
        run: dotnet publish -c ${{ inputs.publish-configuration }} -o wdrop
      - name: Upload a Build Artifact
        uses: actions/upload-artifact@v2.2.2
        with:
          name: ${{ inputs.artifact-name }}
          path: ${{ inputs.project-folder }}/wdrop/**
          if-no-files-found: error
{% endraw %}
~~~

The final workflow.
{:.figcaption}

### Invoking the Workflow
Now that we have a reusable workflow, how do we invoke it? It's pretty easy - almost like invoking an Action. In the code repo, we can add a new workflow that just invokes the centralized workflow we created:

~~~yml
{% raw %}
# file: 'build.yml'
name: Build application

on:
  push:
    branches:
    - main

jobs:
  build-ubuntu:
    name: Build app on Ubuntu
    uses: octodemo/colind-reusable-workflows/.github/workflows/dotnetbuild.yml@main
    with:
      runs-on: ubuntu-latest
      run-tests: false
      project-folder: src
      artifact-name: drop-ubuntu
{% endraw %}
~~~

Invoking the reusable workflow.
{:.figcaption}

We declare a job (`build-ubuntu`) and then use `uses` to specify the job template. We use the format `owner/repo/path@label` to specify the exact location and version of the workflow. Then we use the `with` keyword to specify values for the `inputs`.

## Array Hack
I love being able to pass `stepLists` (or `jobLists`) in Azure Pipelines (see [this doc](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops#parameter-data-types)). This allows you to create generic templates that allow some customization of pre- or post-actions. You could have a template that build an app, allowing some post-build steps to copy files to a location or something along those lines. Unfortunately there is no equivalent in Actions. Even passing arrays of primitives requires a hack.

Let's say you wanted to build the dotnet app on different platforms using a matrix, and you wanted to pass in a list of the platforms. There is no way to pass a list of values into a reusable workflow. But we can use `fromJSON` and pass in a JSON string:

~~~yml
{% raw %}
# file: 'build-template.yml'
nname: Build dotnet application

on:
  workflow_call:
    inputs:
      runs-on:
        description: Platforms to execute on, in format of a string JSON array
        type: string
        default: '["ubuntu-latest"]'
      ...

jobs:
  build:
    name: Build dotnet app
    strategy:
      matrix:
        runs-on: ${{ fromJSON(inputs.runs-on) }}
    
    runs-on: ${{ matrix.runs-on }}
    ...
{% endraw %}
~~~

Creating a JSON string input and parsing it for a matrix.
{:.figcaption}

Then we pass in the JSON string array when we invoke the workflow:

~~~yml
{% raw %}
# file: 'build-app.yml'
name: Build Matrix

on:
  workflow_dispatch:
    
jobs:
  build-ubuntu:
    name: Build app on matrix
    uses: octodemo/colind-reusable-workflows/.github/workflows/build-matrix.yml@main
    with:
      runs-on: '["ubuntu-latest", "windows-latest"]'
      run-tests: false
      project-folder: src
{% endraw %}
~~~

Invoking a workflow with a JSON array parameter.
{:.figcaption}

You can see the value that we pass to `runs-on`: it's a JSON array that has been stringified.

This is definitely hacky, but it's probably the only way to pass a list to a reusable workflow. And it won't work for injecting steps.

## Secrets

If you want to pass secrets to a reusable workflow, you should use the `secrets` keyword. These are really the same as inputs, but instead of being plaintext, they are treated as secrets. Here's an example:

~~~yml
{% raw %}
# file: 'reusable.yml'
name: Deploy

on:
  workflow_call:
    inputs:
      username:
        required: true
        type: string
    secrets:
      token:
        required: true
    
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: ./.github/actions/some-authenticated-action@v1
      with:
        username: ${{ inputs.username }}
        token: ${{ secrets.token }}      
{% endraw %}
~~~

Defining a `secret` input to a reusable workflow.
{:.figcaption}

~~~yml
{% raw %}
# file: 'caller.yml'
name: Deploy

on:
  push:
    
jobs:
  deploy:
    - uses: my-org/my-workflow-repo./.github/workflows/reusable.yml@v1
      with:
        username: ${{ github.actor }}
      secrets:
        token: ${{ github.token }}      
{% endraw %}
~~~

Invoking a reusable workflow with a secret.
{:.figcaption}

You can see how the secret `token` is passed using `secrets` under the `uses` keyword.

**Edit**: GitHub has [added a way to make passing in secrets easier](https://github.blog/changelog/2022-05-03-github-actions-simplify-using-secrets-with-reusable-workflows/) in reusable workflows. Workflows that call reusable workflows in the same organization or enterprise can use the `secrets: inherit` keyword to implicitly pass the secrets. Here's an example:

~~~yml
{% raw %}
# file: 'reusable.yml'
name: Deploy

on:
  workflow_call:
    inputs:
      username:
        required: true
        type: string
    
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: ./.github/actions/some-authenticated-action@v1
      with:
        username: ${{ inputs.username }}
        token: ${{ secrets.token }}      
{% endraw %}
~~~

Defining a `secret` input to a reusable workflow where the caller workflow is using the `secrets: inherit` keyword.
{:.figcaption}

~~~yml
{% raw %}
# file: 'caller.yml'
name: Deploy

on:
  push:
    
jobs:
  deploy:
    - uses: my-org/my-workflow-repo./.github/workflows/reusable.yml@v1
      with:
        username: ${{ github.actor }}
      secrets: inherit
{% endraw %}
~~~

Invoking a reusable workflow with a secret using the `secrets: inherits` keyword.
{:.figcaption}

# Conclusion

Reusable Workflows are a great evolution for GitHub Actions. They can drastically reduce redundancy in your workflows and start paving the way for some centralized templates that can be used to standardize jobs in an org. While they have limitations, they are still powerful tools to add to your toolbelt.

Happy building!
