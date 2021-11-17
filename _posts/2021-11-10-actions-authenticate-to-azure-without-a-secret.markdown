---
layout: post
title: 'GitHub Actions: Authenticate to Azure Without a Secret using OIDC'
date: '2021-11-09 01:22:01'
image: /assets/images/2021/11/oidc/Data_security_27.jpg
description: >
  Authenticating to Azure in GitHub Actions requires a secret for a Service Principal. However, at Universe, GitHub released a new OIDC-based authentication mechanism that eliminates the need for secrets in secure deployments.
tags:
- build
- security
---

1. TOC
{:toc}

> Image from [www.freepik.com](https://www.freepik.com/vectors/business)

# Update: 11/17/2021

In the original version of this post, I extracted the Azure login task to a Composite Action because you need a beta version of the `az cli` for the OIDC to work. As of today, you can use the `@v1` tag of `azure/login` (which has been updated to include OIDC logic) and you do not have to install the beta `az cli`. This makes the Composite Action obsolete - all you have to do now is call the `azure/login` task as before, not passing the secret (assuming you configure the federated credential on the SPN in Azure).

# Problem Statement

Deploying to Azure or other cloud providers from Actions requires that you authenticate to the provider. Not only do you have to authenticate, but the credential you use needs authorization to perform tasks in the cloud platform.

For Azure, this is accomplished by creating a Service Principal (SPN) and then saving the credentials for that SPN to your GitHub repo (or organization) as a secret. The secret is then consumed by the `actions/login` [task](https://github.com/azure/login) to authenticate to Azure before you perform any other steps:

~~~json
{% raw %}
{
  "clientId": "GUID",
  "clientSecret": "some value",
  "tenantId": "GUID",
  "subscriptionId": "GUID"
}
{% endraw %}
~~~

The format of the `AZURE_CREDENTIAL` secret.
{:.figcaption}

The problem with this secret is that it has to be maintained in GitHub. What happens when you rotate the client secret? Deployments will start to fail unless you update the secret. And if you're using the credential across multiple repos, you'll have to update the secret in each place it's used.

# OIDC

To help solve this problem, GitHub has been working with cloud providers to implement [OpenID Connect](https://openid.net/connect/) (OIDC) authentication. This is an established, standard protocol that allows systems to request and receive information about authenticated users. You can read more about the supported providers in the [official documentation](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments).

The idea is that you configure your cloud provider to issue a short-lived token to a specific GitHub repo (optionally scoping to a branch, environment, tag or "any PR"). When you run a workflow, you configure the cloud provider authentication step to request a token for a given security context (SPN for Azure, ARN for AWS for example). If the provider is happy with the request, it issues a token that the step then uses to authenticate the session and the rest of the steps proceed as usual.

For Azure, this means that we can eliminate the _secret_ part of the Azure credential - we still need to know the tenant, subscription and client ID. Since these values are useless without the secret (which you don't need) you don't have to rotate them, or even store them secretly, though storing them as secrets is convenient.

# Sample Action using OIDC

You can take a look at [this repo](https://github.com/colindembovsky/azure-oidc-demo) which contains the code for this post.

For this sample, I wanted to configure two service Principals (`mona-oidc-dev` and `mon-oidc-prod`) that are given access to `oidc-dev` and `oidc-prod` resource groups respectively. I used `dev` and `prod` environments in the repo, and configured the OIDC scoped to the environment, as we'll see later.

I then set up a workflow that used the correct clients for `dev` and `prod`, expecting those to work as advertised. Finally I added a "bad" job that hard-coded the `mona-oidc-dev` client ID and attempted to authenticate to the `prod` environment, just to make sure that the authentication fails.

# Azure Configuration

For the Azure side, I started by creating two service Principals. Nothing special about these, apart from the fact that I have created a **federated credential** that enables the OIDC connection.

## Creating a Service Principal (App Registration)

Navigate to the Active Directory blade in the Azure Portal and click **+Add -> App registration**. Type in the name and URL - these just have to be unique, but can be any value:

![Create a new SPN](/assets/images/2021/11/oidc/create-spn.png){: .center-image }

Create a new SPN.
{:.figcaption}

Once created, click on **Certificates & Secrets** and then on **Federated credentials**. Click **+ Add Credential** to add a new federated credential.

![Configure the OIDC settings](/assets/images/2021/11/oidc/oidc-federation-config.png){: .center-image }

Configure the OIDC settings.
{:.figcaption}

Enter in the `org`, `repo`, `entity type` and additional entity filters if applicable, as well as the `name` and `description` for the credential. You can see at the bottom how Azure constructs the `Subject identifier` for you based on the values you select. Under the hood, when Actions requests a token, it will send a `sub` parameter, and if this matched, Azure will issue a token.

Finally, go back to the **Overview** tab of the app registration and note the **Application (client) ID** and **Directory (tenant) ID**. You'll also need the ID of the subscription you plan to target.

I repeated these steps to create a new SPN (`mona-oidc-prod`) and configured the federated credentials the same way as `mona-oidc-dev` except that I set the `environment` value to `prod` instead of `dev`.

### Authorize the SPN

Don't forget that you have to grant the SPN roles within your subscription(s). If you plan to create resource groups, then assign the SPN the `Contributor` role on your subscription(s). If you want to scope the SPN a little more narrowly, then add it as `Contributor` on one or more resource groups. You can of course use whatever custom roles you need as well.

In my case, I created two resource groups, `oidc-dev` and `oidc-prod` and assigned `mona-oidc-dev` and `mona-oidc-prod` respectively as `Contributors`. This means the `mona-oidc-dev` can only "see" the `oidc-dev` resource group and that `mona-oidc-prod` can only "see" the `oidc-prod` resource group.

# GitHub Configuration

Now we can configure the GitHub side. In **Settings -> Environments** I create two environments: `dev` and `prod`. I then create the following secrets:

Name|Scope|Value
--|--|--
`AZURE_TENANT_ID`|Repo|ID of the AAD tenant
`AZURE_SUBSCRIPTION_ID`|Repo|ID of the target subscription
`AZURE_CLIENT_ID`|`dev` environment|App (client) ID for `mona-oidc-dev`
`AZURE_CLIENT_ID`|`prod` environment|App (client) ID for `mona-oidc-prod`

> Note: I could have specified the `AZURE_SUBSCRIPTION_ID` and `AZURE_TENANT_ID` at environment level if they differed. Most organizations will have separate `dev` and `prod` subscriptions, but usually share a single tenant.

![Configure environments and secrets](/assets/images/2021/11/oidc/secret-config.png){: .center-image }

Configure environments and secrets.
{:.figcaption}

> Note: none of these values constitutes a "secret" value. We have not added a secret for the client secret anywhere - just enough information to tell Azure which context we are going to request a token for.

Of course you can add other environment configuration like approvals and deployment branches. I've left those as empty for this sample.

# The Main Workflow

Here is the workflow that we'll be using:

~~~yml
{% raw %}
# file: '.github/workflows/azure.yml'
name: OIDC Demo

on:
  workflow_dispatch:

permissions:
  id-token: write

jobs:
  dev:
    name: Deploy to Dev
    runs-on: ubuntu-latest
    environment: dev
    steps:
    - uses: actions/checkout@v2
    - name: Azure Login using OIDC
      uses: ./.github/workflows/composite/azure-oidc-login
      with:
        tenant_id: ${{ secrets.AZURE_TENANT_ID }}
        client_id: ${{ secrets.AZURE_CLIENT_ID }}
        subscription_id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
    - name: 'Run az commands'
      run: |
        az account show
        az group list
        pwd

  prod:
    name: Deploy to Prod
    runs-on: ubuntu-latest
    environment: prod
    steps:
    - uses: actions/checkout@v2
    - name: Azure Login using OIDC
      uses: ./.github/workflows/composite/azure-oidc-login
      with:
        tenant_id: ${{ secrets.AZURE_TENANT_ID }}
        client_id: ${{ secrets.AZURE_CLIENT_ID }}
        subscription_id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
    - name: 'Run az commands'
      run: |
        az account show
        az group list
        pwd
        
  bad-prod:
    name: Deploy to Prod using dev
    runs-on: ubuntu-latest
    environment: prod
    steps:
    - uses: actions/checkout@v2
    - name: Azure Login using OIDC
      uses: ./.github/workflows/composite/azure-oidc-login
      with:
        tenant_id: ${{ secrets.AZURE_TENANT_ID }}
        client_id: 84b86cb5-5fbd-4950-88fa-1ab04be41de5
        subscription_id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
    - name: 'Run az commands'
      run: |
        az account show
        az group list
        pwd
{% endraw %}
~~~

The main workflow file.
{:.figcaption}

Notes:
1. We have to specify the `id-token: write` `permission` for this workflow to be able to retrieve and use an OIDC token.
1. There are three jobs: `dev` (which authenticates to `dev`), `prod` (which authenticates to `prod`) and `bad-prod` which uses the `dev` client ID and attempts to get a token for the `prod` environment.
1. Each job uses a composite action (`./.github/workflows/composite/azure-oidc-login`) to authenticate to Azure, passing in the `tenant_id`, `client_id` and `subscription_id`. The values that get passed are the values from the corresponding `environment` that is specified in each job.
1. After the authentication, `az cli` commands are executed, in this case just showing the current subscription (account) and listing the resource groups.
1. The `bad-prod` job is targeting the `prod` environment, so can't use `secrets.AZURE_CLIENT_ID` otherwise it would get the ID for `mona-oidc-prod`. Just for demonstration, I have hard-coded the client ID of `mona-oidc-dev`.

## The Composite Workflow

Why is there a [composite action]({% post_url 2021-09-01-github-composite-actions %}) at all? This is because in order to authenticate to Azure using OIDC, we have to use a beta version of the `az cli`. Rather than install this each time, I extracted this script and the `azure/login` step into a composite that can be reused.

Here is the composite action code:

~~~yml
{% raw %}
# file: '.github/workflows/composite/azure-oidc-login/action.yml
name: OIDC Azure Login

inputs:
  tenant_id:
    description: Azure AAD tenant ID
    require: true
  subscription_id:
    description: Azure subscription ID
    require: true
  client_id:
    description: Azure client ID that has been federated to repo/env/branch/tag
    require: true

runs:
  using: composite
  steps:
  - name: Installing CLI-beta for OpenID Connect
    shell: bash
    run: |
      cd ../..
      CWD="$(pwd)"
      python3 -m venv oidc-venv
      . oidc-venv/bin/activate
      echo "activated environment"
      python3 -m pip install -q --upgrade pip
      echo "started installing cli beta"
      pip install -q --extra-index-url https://azcliprod.blob.core.windows.net/beta/simple/ azure-cli
      echo "***************installed cli beta*******************"
      echo "$CWD/oidc-venv/bin" >> $GITHUB_PATH
  - uses: azure/login@v1.4.0
    name: Log in using OIDC
    with:
      tenant-id: ${{ inputs.tenant_id }}
      client-id: ${{ inputs.client_id }}
      subscription-id: ${{ inputs.subscription_id }}
{% endraw %}
~~~

The composite action to login using OIDC.
{:.figcaption}

Notes:
1. The composite requires three `inputs` - the `tenant_id`, `subscription_id` and `client_id` that we need for the authentication.
1. The first step executes a script to install the beta `az cli`. When the `az cli` is updated to include OIDC features, this step (and the composite action) will no longer be necessary.
1. The second step uses `azure/login@v1.4.0` - this is the release that supports OIDC. It uses the inputs to request a token, and if successful, uses the token to authenticate the remainder of this session (job).

# Running the Workflow

When we run the workflow, it works as expected. The `dev` and `prod` jobs both work correctly. However, the `bad-prod` job fails since the `environment` does not match (remember, this was an attempt to authenticate to the `prod` environment using `mona-oidc-dev` credentials):

![Workflow run](/assets/images/2021/11/oidc/workflow-run.png){: .center-image }

A workflow run.
{:.figcaption}

## Dev Logs

Looking at the logs for `dev` we can see a successful authentication, and we can see that the context only has access to the `oidc-dev` resource group:

![Dev logs](/assets/images/2021/11/oidc/dev-log.png){: .center-image }

Dev logs.
{:.figcaption}

## Prod Logs

Similarly, the `prod` logs show a successful authentication, and we the context only has access to the `oidc-prod` resource group:

![Prod logs](/assets/images/2021/11/oidc/prod-log.png){: .center-image }

Prod logs.
{:.figcaption}

## Bad-prod Logs

Finally, we see that the authentication failed for `bad-prod` since the environment was `prod` but the client ID was for `mona-oidc-dev`, which would have failed the `sub` match in AAD:

![Bad prod logs](/assets/images/2021/11/oidc/bad-prod-log.png){: .center-image }

Bad prod logs.
{:.figcaption}

# Limitations

At present, you cannot specify individual users for the OIDC credential - only repo-level entities. 

Also, if you were thinking you could use a reusable workflow and request tokens from consuming workflows this way, you're out of luck. Even if the `azure/login` step is in a reusable workflow, the `sub` will contain the child repo context. This means that if you use the same reusable workflow in 10 repos, you will have to add all 10 repos as federated identities in the SPN Federated Credentials settings in Azure AAD.

# Conclusion
Authenticating to cloud providers without secrets using OIDC is arguably more secure than having to store secrets. Tokens issues are short-lived, and because teams don't have to store secrets, there is no need to rotate keys. Using OIDC to Azure is fairly simple and does not require a large change to existing workflows.

Happy federating!