---
layout: post
title: Azure Pipeline Variables
date: '2020-02-12 03:35:28'
description: >
  In this post I take a deep dive into Azure Pipeline variables.
tags:
- build
---

1. TOC
{:toc}

I am a big fan of Azure Pipelines. Yes it’s YAML, but once you get over that it’s a fantastic way to represent pipelines as code. It would be tough to achieve any sort of sophistication in your pipelines without variables. There are several types of variables, though this classification is partly mine and pipelines don’t distinguish between these types. However, I’ve found it useful to categorize pipeline variables to help teams understand some of the nuances that occur when dealing with them.

Every variable is really a key:value pair. The key is the name of the variable, and it has a string value. To dereference a variable, simply wrap the key in `$()`. Let’s consider this simple example:

~~~yaml
{% raw %}
    variables:
      name: colin
    
    steps:
    - script: echo "Hello, $(name)!"
{% endraw %}
~~~

This will write “Hello, colin!” to the log.

## Inline Variables

Inline variables are variables that are hard coded into the pipeline YML file itself. Use these for specifying values that are not sensitive and that are unlikely to change. A good example is an image name: let’s imagine you have a pipeline that is building a Docker container and pushing that container to a registry. You are probably going to end up referencing the image name in several steps (such as tagging the image and then pushing the image). Instead of using a value in-line in each step, you can create a variable and use it multiple times. This keeps to the DRY (Do not Repeat Yourself) principle and ensures that you don’t inadvertently misspell the image name in one of the steps. In the following example, we create a variable called “imageName” so that we only have to maintain the value once rather than in multiple places:

~~~yaml
{% raw %}
    trigger:
    - master
    
    pool:
      vmImage: ubuntu-latest
    
    variables:
      imageName: myregistry/api-image
    
    steps:
    - task: Docker@2
      displayName: Build an image
      inputs:
        repository: $(imageName)
        command: build
        Dockerfile: api/Dockerfile
    
    - task: Docker@2
      displayName: Push image
      inputs:
        containerRegistry: $(ACRRegistry)
        repository: $(imageName)
        command: push
        tags: $(Build.BuildNumber)
{% endraw %}
~~~

Note that you obviously you cannot create "secret" inline variables. If you need a variable to be secret, you’ll have to use pipeline variables, variable groups or dynamic variables.

## Predefined Variables

There are several predefined variables that you can reference in your pipeline. Examples are:

- Source branch: “Build.SourceBranch”
- Build reason: “Build.Reason”
- Artifact staging directory: “Build.ArtifactStagingDirectory”

You can find a full list of predefined variables [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml).

## Pipeline Variables

Pipeline variables are specified in Azure DevOps in the pipeline UI when you create a pipeline from the YML file. These allow you to abstract the variables out of the file. You can specify defaults and/or mark the variables as "secrets" (we’ll cover secrets a bit later). This is useful if you plan on triggering the pipeline manually and want to set the value of a variable at queue time.

One thing to note: if you specify a variable in the YML variables section, you cannot create a pipeline variable with the same name. If you plan on using pipeline variables, you must **not** specify them in the "variables" section!

When should you use pipeline variables? These are useful if you plan on triggering the pipeline manually and want to set the value of a variable at queue time. Imagine you sometimes want to build in “DEBUG” and other times in “RELEASE”: you could specify “buildConfiguration” as a pipeline variable when you create the pipeline, giving it a default value of “debug”:

<!--kg-card-begin: html-->[![image](/assets/images/files/a42362d3-34a7-4a89-a075-00632243cbcb.png "image")](/assets/images/files/704856b2-fdc7-4dc5-b318-6ef8a5af42a6.png)<!--kg-card-end: html-->

If you specify “Let users override this value when running this pipeline” then users can change the value of the pipeline when they manually queue it. Specifying “Keep this value secret” will make this value a secret (Azure DevOps will mask the value).

Let's look at a simple pipeline that consumes the pipeline variable:

~~~yaml
{% raw %}
    name: 1.0$(Rev:.r)
    
    trigger:
    - master
    
    pool:
      vmImage: ubuntu-latest
      
    jobs:
    - job: echo
      steps:
      - script: echo "BuildConfiguration is $(buildConfiguration)"
{% endraw %}
~~~

Running the pipeline without editing the variable produces the following log:

<!--kg-card-begin: html-->[![image](/assets/images/files/03d237c0-0360-47e9-927e-040dc5def391.png "image")](/assets/images/files/e5016918-35d6-4037-a237-2669c32a66c4.png)<!--kg-card-end: html-->

If the pipeline is not manually queued, but triggered, any pipeline variables default to the value that you specify in the parameter when you create it.

Of course if we update the value when we queue the pipeline to “release”, of course the log reflects the new value:

<!--kg-card-begin: html-->[![image](/assets/images/files/ff8674b4-bfcf-49d0-83fd-5ae5a906ded8.png "image")](/assets/images/files/b47f930b-20dd-484a-ba87-3cf9078655a6.png)<!--kg-card-end: html-->

Referencing a pipeline variable is exactly the same as referencing an inline variable – once again, the distinction is purely for discussion.

## Secrets

At some point you’re going to want a variable that isn’t visible in the build log: a password, an API Key etc. As I mentioned earlier, inline variables are never secret. You must mark a pipeline variable as secret in order to make it a secret, or you can create a dynamic variable that is secret.

"Secret" in this case just means that the value is masked in the logs. It is still possible to expose the value of a secret if you really want to. A malicious pipeline author could “echo” a secret to a file and then open the file to get the value of the secret.

All is not lost though: you can put controls in place to ensure that nefarious developers cannot simply run updated pipelines – you should be using Pull Requests and Branch Policies to review changes to the pipeline itself (an advantage to having pipelines as code). The point is, you still need to be careful with your secrets!

## Dynamic Variables and Logging Commands

Dynamic variables are variables that are created and/or calculated at run time. A good example is using the “az cli” to retrieve the connection string to a storage account so that you can inject the value into a web.config. Another example is dynamically calculating a build number in a script.

To create or set a variable dynamically, you can use [logging commands](https://docs.microsoft.com/en-us/azure/devops/pipelines/scripts/logging-commands?view=azure-devops&tabs=bash). Imagine you need to get the username of the current user for use in subsequent steps. Here’s how you can create a variable called “currentUser” with the value:

~~~yaml
{% raw %}
    - script: |
        curUser=$(whoami)
        echo "##vso[task.setvariable variable=currentUser;]$curUser"
{% endraw %}
~~~

When writing bash or PowerShell commands, don’t confuse “$(var)” with “$var”. “$(var)” is interpolated by Azure DevOps when the step is executed, while “$var” is a bash or PowerShell variable. I often use “env” to create environment variables rather than dereferencing variables inline. For example, I could write:

~~~yaml
{% raw %}
    - script: echo $(Build.BuildNumber)
{% endraw %}
~~~

but I can also use environment variables:

~~~yaml
{% raw %}
    - script: echo $buildNum
      env:
        buildNum: $(Build.BuildNumber)
{% endraw %}
~~~

This may come down to personal preference, but I’ve avoided confusion by consistently using env for my scripts!

To make the variable a secret, simple add “issecret=true” into the logging command:

~~~yaml
{% raw %}
    echo "##vso[task.setvariable variable=currentUser;issecret=true]$curUser"
{% endraw %}
~~~

You could do the same thing using PowerShell:

~~~yaml
{% raw %}
    - powershell: |
        Write-Host "##vso[task.setvariable variable=currentUser;]$env:UserName"
{% endraw %}
~~~

Note that there are two flavors of PowerShell: “powershell” is for Windows and “pwsh” is for PowerShell Core which is cross-platform (so it can run on Linux and Mac!).

One special case of a dynamic variable is a calculated build number. For that, calculate the build number however you need to and then use the “build.updatebuildnumber” logging command:

~~~yaml
{% raw %}
    - script: |
        buildNum=$(...) # calculate the build number somehow
        echo "##vso[build.updatebuildnumber]$buildNum"
{% endraw %}
~~~

Other logging commands are documented [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/scripts/logging-commands?view=azure-devops&tabs=bash#build-commands).

## Variable Groups

Creating inline variables is fine for values that are not sensitive and that are not likely to change very often. Pipeline variables are useful for pipelines that you want to trigger manually. But there is another option that is particularly useful for multi-stage pipelines (we'll cover these in more detail later).

Imagine you have a web application that connects to a database that you want to build and then push to DEV, QA and Prod environments. Let's consider just one config setting - the database connection string. Where should you store the value for the connection string? Perhaps you could store the DEV connection string in source control, but what about QA and Prod? You probably don't want those passwords stored in source control.

You could create them as pipeline variables - but then you'd have to prefix the value with an environment or something to distinguish the QA value from the Prod value. What happens if you add in a STAGING environment? What if you have other settings like API Keys? This can quickly become a mess.

This is what Variable Groups are designed for. You can find variable groups in the “Library" hub in Azure DevOps:

<!--kg-card-begin: html-->[![image](/assets/images/files/623a4b38-40b4-41ba-b9da-3352a1ab04a1.png "image")](/assets/images/files/a4887fed-9394-4f2f-8742-1ffd05c8bc26.png)<!--kg-card-end: html-->

The image above shows two variable groups: one for DEV and one for QA. Let's create a new one for Prod, specifying the same variable name (“ConStr”) but this time entering in the value for Prod:

<!--kg-card-begin: html-->[![image](/assets/images/files/d4c5c376-a471-4aab-8f03-7452eaa9a112.png "image")](/assets/images/files/faa615fb-4f15-4eb2-bc61-d5f22f370e59.png)<!--kg-card-end: html-->

Security is beyond the scope of this post- but you can specify who has permission to view/edit variable groups, as well as which pipelines are allowed to consume them. You can of course mark any value in the variable group as secret by clicking the padlock icon next to the value.

The trick to making variable groups work for environment values is to keep the names the same in each variable group. That way the only setting you need to update between environments is the variable group name. I suggest getting the pipeline to work completely for one environment, and then “Clone” the variable group - that way you're assured you're using the same variable names.

### KeyVault Integration

You can also integrate variable groups to Azure KeyVaults. When you create the variable group, instead of specifying values in the variable group itself, you connect to a KeyVault and specify which keys from the vault should be synchronized when the variable group is instantiated in a pipeline run:

<!--kg-card-begin: html-->[![image](/assets/images/files/766709c4-081e-4bed-8251-22f9e5fd6613.png "image")](/assets/images/files/7e8565f1-a6a3-4836-8348-80a83c4d46fe.png)<!--kg-card-end: html-->
### Consuming Variable Groups

Now that we have some variable groups, we can consume them in a pipeline. Let's consider this pipeline:

~~~yaml
{% raw %}
    trigger:
    - master
    
    pool:
      vmImage: ubuntu-latest
      
    jobs:
    - job: DEV
      variables:
      - group: WebApp-DEV
      - name: environment
        value: DEV
      steps:
      - script: echo "ConStr is $(ConStr) in enviroment $(environment)"
    
    - job: QA
      variables:
      - group: WebApp-QA
      - name: environment
        value: QA
      steps:
      - script: echo "ConStr is $(ConStr) in enviroment $(environment)"
    
    - job: Prod
      variables:
      - group: WebApp-Prod
      - name: environment
        value: Prod
      steps:
      - script: echo "ConStr is $(ConStr) in enviroment $(environment)"
{% endraw %}
~~~

When this pipeline runs, we’ll see the DEV, QA and Prod values from the variable groups in the corresponding jobs.

Notice that the format for inline variables alters slightly when you have variable groups: you have to use the “- name/value” format.

## Variable Templates

There is another type of template that can be useful - if you have a set of inline variables that you want to share across multiple pipelines, you can create a template. The template can then be referenced in multiple pipelines:

~~~yaml
{% raw %}
    # templates/variables.yml
    variables:
    - name: buildConfiguration
      value: debug
    - name: buildArchitecture
      value: x64
    
    # pipelineA.yml
    variables:
    - template: templates/variables.yml
    steps:
    - script: build x ${{ variables.buildArchitecture }} ${{ variables.buildConfiguration }}
    
    # pipelineB.yml
    variables:
    - template: templates/variables.yml
    steps:
    - script: echo 'Arch: ${{ variables.buildArchitecture }}, config ${{ variables.buildConfiguration }}'
{% endraw %}
~~~

## Precedence and Expansion

Variables can be defined at various scopes in a pipeline. When you define a variable with the same name at more than one scope, you need to be aware of the precedence. You can read the documentation on precedence [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch#expansion-of-variables).

You should also be aware of _when_ variables are expanded. They are expanded at the beginning of the run, as well as before each step. This example shows how this works:

~~~yaml
{% raw %}
    jobs:
    - job: A
      variables:
        a: 10
      steps:
        - bash: |
            echo $(a) # This will be 10
            echo '##vso[task.setvariable variable=a]20'
            echo $(a) # This will also be 10, since the expansion of $(a) happens before the step
        - bash: echo $(a) # This will be 20, since the variables are expanded just before the step
{% endraw %}
~~~

## Conclusion

Azure Pipelines variables are powerful – and with great power comes great responsibility! Hopefully you understand variables and some of their gotchas a little better now. There’s another topic that needs to be covered to complete the discussion on variables – _parameters_. I’ll cover parameters in a follow up post.

For now – happy building!

