---
layout: post
title: On Demand Ephemeral Self-Hosted Runners
date: '2021-10-26 01:22:01'
image: /assets/images/2021/10/self-hosted-runners/boom.jpeg
description: >
  Do you need to deploy to private VNets using GitHub Actions, but don't want to have to keep self-hosted runners running all the time? In this post I show you how you can use Ephemeral Runners to create on-demand self-hosted runners.
tags:
- actions
- build
---

1. TOC
{:toc}

# Self-Hosted Runners

GitHub [hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners) are great if you need to run workflows that don't require special tools or access to protected resources. If you have specific tool requirements, you can install tools as steps within a workflow. However, if custom tool installs take too long, or you have other build/deploy requirements (like license files for 3rd party libraries), you may want to create custom images (containers or VMs) that you can run [self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners) on. Obviously you'll need self-hosted runners to deploy to private networks.

But self-hosted runners mean overhead - you have to maintain them as well as pay for the compute. One technique you could use is to [autoscale your self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/autoscaling-with-self-hosted-runners). However, this requires that you have AKS or use the Terraform/AWS method. What if you want something a little more light-weight?

# Using WebHooks

When a workflow is queued, GitHub publishes an event `workflow_job` (docs [here](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#workflow_job)). Initially when thinking about this scenario, I explored creating a workflow that would trigger off this event and spin up a self-hosted runner.

Unfortunately, most, but not _all_ GitHub events trigger workflows. You can create a webhook to listen for `workflow_job` and then call a REST API - but then you'd have to host something capable of processing the `POST` from the webhook, which could then turn around and `POST` to the `repository_webhook` event of a workflow. Sounds like more overhead to manage!

Instead of doing this, I realized that I could create a reusable workflow to spin up (and another to tear down) a self-hosted runner. In this manner, I'd be spinning up a self-hosted runner _on demand_ and tearing it down afterwards to save overhead and compute.

# On-Demand Self-Hosted Runners

In this post, I'll walk you through this scenario. The idea is as follows:

1. When a workflow runs, execute a job on a hosted runner that spins up a self-hosted runner using Azure Container Instances (ACI) connected to a specific VNet, registering the runner as [ephemeral](https://docs.github.com/en/actions/hosting-your-own-runners/autoscaling-with-self-hosted-runners#using-ephemeral-runners-for-autoscaling)
1. Execute the "real" work, targeting the self-hosted runner that was just created
1. When the job completes, the runner unregisters from the repo (since it was created as an ephemeral runner)
1. Execute a clean-up job that deletes the ACI

In this manner, you get the experience of a hosted runner, but the "real" work is performed on a self-hosted runner that is spun up on-demand in your private VNet. You sacrifice a little bit of time (it takes some time to spin up the ACI) but in return you decrease overhead, since the ACI only lives just long enough to execute the work - there's nothing to manage after the job completes.

> Note: You also have to maintain the container image, but if you're not frequently messing with the tools, this is usually not a lot of overhead.

> Note: The GitHub PAT required by the self-hosted runner is **VISIBLE on the config of the ACI in the Azure Portal or `az cli`**. Ensure that you set up appropriate RBAC to prevent leaking this credential!

Code for this post can be found in [this repo](https://github.com/colindembovsky/scaling-self-hosted-aci).

## Prerequisites

In order for this to work, you'll need a container image that, when started, will register an ephemeral runner. That image should then be customized to install any further custom build or deploy tools that you may need. You can see this [example repo](https://github.com/colindembovsky/github-actions-runner-container) that builds a container image that registers a self-hosted runner when the container is started. That repo includes a workflow that will build the `Dockerfile` and publish the image to GitHub Container Registry (GHCR).

Secondly, you need to create a resource group for housing the ACI instances and to create an SPN that has permissions to create resources in the resource group.

Finally, you need to get the ID of the subnet that you want to connect your runner to. If you're deploying to private VNets, this is an essential step.

> Note: You could customize the code so that the subnet is not required - this will just spin an ACI that is not connected to any specific VNet. You'd then have a self-hosted runner running on your custom image, but it won't be connected to any private VNets.

## Reusable Workflows

There are two [reusable workflows](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows) in this scenario: one to spin up an ACI that connects a self-hosted runner to the invoking workflow's repository. The second deletes the ACI. We don't have to worry about unregistering the runner, since we'll spin the runner using the `--ephemeral` switch, which automatically unregisters the runner after it has completed a single job.

Both workflows execute using hosted runners. The hosted runners simply spin up (and tear down) the ephemeral self-hosted runner: but it is the self-hosted runner that is doing the "real" work.

### Deploy Ephemeral Runner

The code for the deployment workflow is as follows:

~~~yml
{% raw %}
# file: 'deploy-ephemeral-runner.yml'
name: Deploy Ephemeral Runner

on:
  workflow_call:
    inputs:
      rg_name:
        type: string
        description: Name of RG to deploy to
        default: cd-ephemeral
      location:
        type: string
        description: Location for ACI
        default: southcentralus
      subnet_id:
        type: string
        description: Subnet to create ACI on
        default: /subscriptions/f12d732d-4a47-4edc-a11b-d6dc6909ddbe/resourceGroups/cd-ephemeral/providers/Microsoft.Network/virtualNetworks/private-vnet/subnets/default
      aci_prefix:
        type: string
        description: Prefix for ACI name
        default: cd-shrunner-aci
      runner_image:
        type: string
        description: Image of runner container
        default: ghcr.io/colindembovsky/ubuntu-actions-runner:6d7a59dfa95ec094a5fa8292bad01158c374e3ad
      labels:
        type: string
        description: Comma-separated list of labels to apply to the runner
        required: true
        
    secrets:
      azure_creds:
        description: Azure credentials
        required: true
      repo_pat:
        description: PAT with `repo` permissions
        required: true

jobs:
  deploy_runner:
    name: Deploy Ephemeral Runner
    runs-on: ubuntu-latest
    steps:
    - name: Login to Azure
      uses: azure/login@v1
      with:
        creds: ${{ secrets.azure_creds }}
    
    - name: Create runner
      run: |
        az container create \
          -g ${{ inputs.rg_name }} -n ${{ inputs.aci_prefix }}-${{ github.run_id }} \
          --image ${{ inputs.runner_image }} --restart-policy Never \
          --subnet ${{ inputs.subnet_id }} \
          --environment-variables \
            RUNNER_REPOSITORY_URL=https://github.com/${{ github.repository }} \
            GITHUB_TOKEN=${{ secrets.repo_pat }} \
            RUNNER_OPTIONS="--ephemeral" \
            RUNNER_LABELS=${{ inputs.labels }} \
            RUNNER_NAME=${{ inputs.aci_prefix }}-${{ github.run_id }}
{% endraw %}
~~~

Workflow to deploy an ephemeral self-hosted runner to ACI.
{:.figcaption}

Notes:
1. We specify the `workflow_call` trigger in the `on` section to indicate that this is a reusable workflow.
1. We include an `inputs` for specifying the `rg_name` (resource group name), `location`, `subnet_id`, `aci_prefix` and `runner_image`. Most inputs are self-explanatory - we'll append the `run_id` to the `aci_prefix` to ensure a unique ACI per workflow run.
1. We need to pass in two `secrets`: `azure_creds` (to authenticate to Azure) as well as `repo_pat` which is a GitHub Personal Access Token (PAT) that has `repo` priveledges. We need this to register the runner to the repo.
1. The first step authenticates to Azure.
1. The second step uses `az container create` to create the ACI using the inputs provided. A few callouts: we set the `--restart-policy` to `Never` since we don't want the container to restart when the runner unregisters after completing a job. We attache the ACI to the specified subnet via the `subnet` argument. Finally, we pass in `--ephemeral` to mark the runner as ephemeral.

### Delete ACI

To delete the ACI after the runner has unregistered, we can use this reusable workflow:

~~~yml
{% raw %}
# file: 'delete-aci.yml'
name: Delete ACI

on:
  workflow_call:
    inputs:
      rg_name:
        type: string
        description: Name of RG to deploy to
        default: cd-ephemeral
      aci_prefix:
        type: string
        description: Prefix for ACI name
        default: cd-shrunner-aci
        
    secrets:
      azure_creds:
        description: Azure credentials
        required: true

jobs:
  delete_aci:
    name: Delete ACI
    runs-on: ubuntu-latest
    steps:
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.azure_creds }}
    
    - name: Delete ACI
      run: |
        az container delete -g ${{ inputs.rg_name }} -n ${{ inputs.aci_prefix }}-${{ github.run_id }} --yes
{% endraw %}
~~~

Workflow to delete the ACI.
{:.figcaption}

Notes:
1. We pass in the `rg_name` and `aci_prefix` just like we did for the `Deploy ACI` workflow.
1. We pass in the `azure_creds` to authenticate to Azure.
1. After logging in to Azure, we invoke `az container delete` with `--yes` to delete the ACI completely.

> Note: Just a reminder: at this stage, the runner has unregistered itself and the container is no longer running. We're just cleaning up the ACI to ensure we are not billed for compute.

## The Main Workflow

Now that we have automation to spin up the ACI and tear it down, we can incorporate these jobs into our "main" workflow:

~~~yml
{% raw %}
# file: 'work.yml'
name: Do Some Work

on:
  workflow_dispatch:

jobs:
  # deploy a runner for the job
  deploy_runner:
    uses: colindembovsky/scaling-self-hosted-aci/.github/workflows/deploy-ephemeral-runner.yml@main
    with:
      labels: uniqueString
    secrets:
      azure_creds: ${{ secrets.AZURE_CREDENTIALS }}
      repo_pat: ${{ secrets.REPO_PAT }}
  
  # do the real work
  build:
    needs: deploy_runner
    runs-on: [ self-hosted, uniqueString ]

    steps:
      - uses: actions/checkout@v2
      - name: Simulate some work
        run: |
          sleep 10

  delete_aci:
    needs: build
    if: ${{ always() }}
    uses: colindembovsky/scaling-self-hosted-aci/.github/workflows/delete-aci.yml@main
    secrets:
      azure_creds: ${{ secrets.AZURE_CREDENTIALS }}
{% endraw %}
~~~

The main workflow.
{:.figcaption}

Notes:
1. We invoke the deploy runner workflow which spins up a self-hosted runner in an ACI connected to our private VNet. To make sure that our job targets the correct runner, we add a `labels` value of `uniqueString`.
1. In the `build` job, we target any runner that has labels `self-hosted` and `uniqueString` (there should only be the one).
1. Steps executed here run on the self-hosted runner, that is connected to our private VNet, so it should be able to access resource on this VNet and any peered VNets.
1. The `delete_aci` job invokes the reusable workflow to clean up the ACI after the job complets. We add `if: ${{ always() }}` to ensure that the job executes even if the `build` job fails.
1. We use `needs` to specify the ordering: first create the ACI hosting the runner, then execute the `build` and then only delete the ACI.

### Secrets

In order for this to work, you'll need to configure [Azure Credentials](https://github.com/marketplace/actions/azure-login) to the SPN that can create resources in the resource group and that has permissions to add interfaces to the subnet. You'll also need to generate a GitHub PAT that has `repo` priveledges.

## Analyzing the Runs

If you run the above workflow, you'll have a small window of time where you can see the self-hosted runner (navigate to **Settings->Actions->Runners** on the repo):

![Self-hosted runner registered after the ACI spins up](/assets/images/2021/10/self-hosted-runners/runner-registered.png){: .center-image }

Self-hosted runner registered after the ACI spins up.
{:.figcaption}

When the job completes, the runner unregisters itself and is no longer listed. Similarly, if you look at your Azure subscription, you'll see the ACI spin up. If you click **Containers** and check the logs for the container, you'll see the runner register and wait for jobs:

![Logs in ACI after spinning up](/assets/images/2021/10/self-hosted-runners/runner-aci-ready.png){: .center-image }

Logs in ACI after spinning up.
{:.figcaption}

Once the job completes, the runner automatically unregisters. Again, looking at the logs you'll see this operation:

![Logs after completing the job](/assets/images/2021/10/self-hosted-runners/runner-unregistered.png){: .center-image }

Logs after completing the job.
{:.figcaption}

The ACI is then removed to complete the workflow.

![Completed workflow](/assets/images/2021/10/self-hosted-runners/workflow-completed.png){: .center-image }

The completed workflow. The "work" was to `sleep 120`.
{:.figcaption}

Looking at the jobs, I recorded spinning the ACI took about 90 - 180 seconds. Tear down took approximately 30 seconds. All in all, we're adding about 2 minutes total time to the job, but we don't have any infrastructure to manage once the jobs complete. Not too bad to get an "on-demand" self-hosted agent!

# Conclusion

Hosted runners provide a very low overhead mechanism for running Actions. However, there are times when you need custom images or when you're targeting private VNets, and in those situations you need to run a self-hosted agent. But this requires that you manage a container runtime (like Kubernetes) or manage VMs. However, if you are looking to deploy to private VNets or run jobs on custom images without having to manage long-lived runners, then using ACI to host ephemeral runners is a viable "on-demand" solution.

Happy building!
