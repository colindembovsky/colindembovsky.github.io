---
layout: post
title: Azure Pipeline Parameters
date: '2020-02-27 07:35:44'
description: >
  In this post I dive into parameters for Azure Pipelines.
tags:
- build
---

1. TOC
{:toc}

In a [previous post](/azure-pipeline-variables), I did a deep dive into Azure Pipeline variables. That post turned out to be longer than I anticipated, so I left off the topic of parameters until this post.

## Type: Any

If we look at the YML schema for variables and parameters, we’ll see this definition:

~~~yaml
{% raw %}
    variables: { string: string }
    
    parameters: { string: any }
{% endraw %}
~~~

Parameters are essentially the same as variables, with the following important differences:

{% raw %}
- Parameters are dereferenced using “${{}}” notation
- Parameters can be complex objects
- Parameters are expanded at queue time, not at run time
- Parameters can only be used in templates (you cannot pass parameters to a pipeline, only variables)
{% endraw %}

Parameters allow us to do interesting things that we cannot do with variables, like if statements and loops. Before we dive in to some examples, let’s consider _variable dereferencing_.

## Variable Dereferencing

The [official documentation](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch#understand-variable-syntax) specifies three methods of dereferencing variables: macros, template expressions and runtime expressions:

{% raw %}
- Macros: this is the “$(var)” style of dereferencing
- Template parameters use the syntax “${{ parameter.name }}”
- Runtime expressions, which have the format “$[variables.var]”
{% endraw %}

{% raw %}
In practice, the main thing to bear in mind _is when the value is injected_. “$()” variables are expanded at runtime, while “${{}}” parameters are expanded at _compile_ time. Knowing this rule can save you some headaches.
{% endraw %}

The other notable difference is left vs right side: variables can only expand on the right side, while parameters can expand on left or right side. For example:

~~~yaml
{% raw %}
    # valid syntax
    key: $(value)
    key: $[variables.value]
    ${{ parameters.key }} : ${{ parameters.value }}
    
    # invalid syntax
    $(key): value
    $[variables.key]: value
{% endraw %}
~~~

Here's a real-life example from a [TailWind Traders](http://pipelinehttps://github.com/10thmagnitude/TailwindTraders-Backend/blob/master/Pipeline/azure-pipeline.yaml) I created. In this case, the repo contains several microservices that are deployed as Kubernetes services using Helm charts. Even though the code for each microservice is different, the _deployment_ for each is identical, except for the path to the Helm chart and the image repository.

Thinking about this scenario, I wanted a template for deployment steps that I could parameterize. Rather than copy the entire template, I used a “for” expression to iterate over a map of complex properties. For each service deployment, I wanted:

- serviceName: The path to the service Helm chart
- serviceShortName: Required because the deployment requires two steps: “bake” the manifest, and then “deploy” the baked manifest. The “deploy” task references the output of the “bake” step, so I needed a name that wouldn't collide as I expanded it multiple times in the “for” loop

Here's a snippet of the template steps:

~~~yaml
{% raw %}
    # templates/step-deploy-container-service.yml
    parameters:
      serviceName: '' # product-api
      serviceShortName: '' # productapi
      environment: dev
      imageRepo: '' # product.api
      ...
      services: []
    
    steps:
    - ${{ each s in parameters.services }}:
      - ${{ if eq(s.skip, 'false') }}:
        - task: KubernetesManifest@0
          displayName: Bake ${{ s.serviceName }} manifest
          name: bake_${{ s.serviceShortName }}
          inputs:
            action: bake
            renderType: helm2
            releaseName: ${{ s.serviceName }}-${{ parameters.environment }}
            ...
        - task: KubernetesManifest@0
          displayName: Deploy ${{ s.serviceName }} to k8s
          inputs:
            manifests: $(bake_${{ s.serviceShortName }}.manifestsBundle)
            imagePullSecrets: $(imagePullSecret)
{% endraw %}
~~~

Here's a snippet of the pipeline that references the template:

~~~yaml
{% raw %}
    ...
      - template: templates/step-deploy-container-service.yml
        parameters:
          acrName: $(acrName)
          environment: dev
          ingressHost: $(IngressHost)
          tag: $(tag)
          autoscale: $(autoscale)
          services:
          - serviceName: 'products-api'
            serviceShortName: productsapi
            imageRepo: 'product.api'
            skip: false
          - serviceName: 'coupons-api'
            serviceShortName: couponsapi
            imageRepo: 'coupon.api'
            skip: false
          ...
          - serviceName: 'rewards-registration-api'
            serviceShortName: rewardsregistrationapi
            imageRepo: 'rewards.registration.api'
            skip: true
{% endraw %}
~~~

In this case, “services” could not have been a variable since variables can only have “string” values. Hence I had to make it a parameter.

## Parameters and Expressions

There are a number of expressions that allow us to create more complex scenarios, especially in conjunction with parameters. The example above uses both the “each” and the “if” expressions, along with the boolean function “eq”. Expressions can be used to loop over steps or ignore steps (as an equivalent of setting the “condition” property to “false”). Let's look at an example in a bit more detail. Imagine you have this template:

~~~yaml
{% raw %}
    # templates/steps.yml
    parameters:
      services: []
    
    steps:
    - ${{ each s in parameters.services }}:
      - ${{ if eq(s.skip, 'false') }}:
        - script: echo 'Deploying ${{ s.name }}'
{% endraw %}
~~~

Then if you specify the following pipeline:

~~~yaml
    jobs:
    - job: deploy
      - steps: templates/steps.yml
        parameters:
          services:
          - name: foo
            skip: false
          - name: bar
            skip: true
          - name: baz
            skip: false
~~~

you should get the following output from the steps:

<!--kg-card-begin: html--><font face="Courier New">Deploying foo<br>
Deploying baz</font><!--kg-card-end: html-->

Parameters can also be used to inject steps. Imagine you have a set of steps that you want to repeat with different parameters - except that in some cases, a slightly different middle step needs to be executed. You can create a template that has a parameter called “middleSteps” where you can pass in the middle step(s) as a parameter!

~~~yaml
{% raw %}
    # templates/steps.yml
    parameters:
      environment: ''
      middleSteps: []
    
    steps:
    - script: echo 'Prestep'
    - ${{ parameters.middleSteps }}
    - script: echo 'Post-step'
    
    # pipelineA
    jobs:
    - job: A
      - steps: templates/steps.yml
        parameters:
          middleSteps:
          - script: echo 'middle A step 1'
          - script: echo 'middle A step 2'
    
    # pipelineB
    jobs:
    - job: B
      - steps: templates/steps.yml
        parameters:
          middleSteps:
          - script: echo 'This is job B middle step 1'
          - task: ... # some other task
          - task: ... # some other task
{% endraw %}
~~~

For a real world example of this, see this [template file](https://github.com/10thmagnitude/MLOpsDemo/blob/master/templates/job-train-model.yml). This is a demo where I have two scenarios for machine learning: a manual training process and an AutoML training process. The pre-training and post-training steps are the same, but the training steps are different: the template reflects this scenario by allowing me to pass in different “TrainingSteps” for each scenario.

## Extends Templates

Passing steps as parameters allows us to create what Azure DevOps calls “[extends templates](https://docs.microsoft.com/en-us/azure/devops/pipelines/security/templates?view=azure-devops#use-extends-templates)”. These provide rails around what portions of a pipeline can be customized, allowing template authors to inject (or remove) steps. The following example from the documentation demonstrates this:

~~~yaml
{% raw %}
    # template.yml
    parameters:
    - name: usersteps
      type: stepList
      default: []
    steps:
    - ${{ each step in parameters.usersteps }}:
      - ${{ each pair in step }}:
        ${{ if ne(pair.key, 'script') }}:
          ${{ pair.key }}: ${{ pair.value }}
    
    # azure-pipelines.yml
    extends:
      template: template.yml
      parameters:
        usersteps:
        - task: MyTask@1
        - script: echo This step will be stripped out and not run!
        - task: MyOtherTask@2
{% endraw %}
~~~

## Conclusion

Parameters allow us to pass and manipulate complex objects, which we are unable to do using variables. They can be combined with expressions to create complex control flow. Finally, parameters allow us to control how a template is customized using extends templates.

Happy parameterizing!

