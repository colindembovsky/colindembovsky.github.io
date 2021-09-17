---
layout: post
title: az devops cli like a boss
date: '2020-07-20 19:59:14'
description: >
  One of the best features of Azure DevOps is the extensive API. However, while having a REST API is great, interacting with a service at HTTP level can be frustrating. In this post, I examine the az devops cli using 10 practical examples.
tags:
- automation
---

1. TOC
{:toc}

One of the best features of Azure DevOps is the [extensive API](https://docs.microsoft.com/en-us/rest/api/azure/devops/?view=azure-devops-rest-6.0). However, while having a REST API is great, interacting with a service at HTTP level can be frustrating.

Azure itself has an [extensive API](https://docs.microsoft.com/en-us/rest/api/azure/), and the API has been wrapped into an easy to use cross-platform command line interface (CLI) called `az`. Fortunately, there is an extension to `az` for interacting with Azure DevOps called `az devops`.

In this post I'll walk through installing the cli, some basics for using the cli effectively and then 10 practical examples of how to use it:

1. Creating a Team Project
2. Managing Security Groups
3. Determining if a Git Repo Exists
4. Creating a Git Repo and Importing an External Repo
5. (Bonus) Automating Git Commands After Cloning
6. Deleting a Git Repo
7. Creating an ARM Service Endpoint
8. Creating and Deleting (YML) Environments using invoke
9. Creating Variable Groups
10. Creating YML Pipelines

## Installing az and az devops

To install `az` you can follow [these instructions](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest). Once you've installed `az` you can install the `devops` extension by following [these instructions](https://github.com/Azure/azure-devops-cli-extension#quick-start) (all you really have to do is type `az extension add --name azure-devops`).

## -h is your friend

The `-h` switch (help) is your friend. I have often discovered new subcommands by using the switch, and of course for each command you need the help to figure out all of the args you need to pass. For example, `az devops project create -h` prints out all the args needed to create a new Azure DevOps Team Project.

## Organization

When you run `az devops` commands, you'll need to specify the organization (and often Team Project) that you want to run commands against. Supposedly, you can set a default using `az devops configure defaults organization=https://dev.azure.com/myOrg` but I could not get that to work. Instead, I put the org URL into a variable and use the `--org $orgUrl` argument for every command.

## JMESPath

The other argument that you'll need to want to know is the `--query` argument. This allows you to specify JSON queries against the results of a command to extract certain information. JMESPath is at once powerful and frustrating. You can find documentation on this query language [here](https://jmespath.org/).

You'll also want to use the `-o` parameter to specify the output. For scripts, using `-o tsv` (table separated values) will give you plain text results that you can assign to variables for use further down in your scripts.

For example, if you want to determine if a Team Project already exists, you can use the `az devops project list` command: but it will return an array of objects which you'll have to try to parse:

~~~bash
{% raw %}
    /home/colin/repos/foo [master ≡]> az devops project list --org $orgUrl
    {
      "continuationToken": null,
      "value": [
        {
          "abbreviation": null,
          "defaultTeamImageUrl": null,
          "description": null,
          "id": "<redacted>",
          "lastUpdateTime": "2020-07-17T20:54:47.090000+00:00",
          "name": "Project1",
          "revision": 622,
          "state": "wellFormed",
          "url": "https://dev.azure.com/<redacted>/_apis/projects/<redacted>",
          "visibility": "private"
        },
        {
          "abbreviation": null,
          "defaultTeamImageUrl": null,
          "description": null,
          "id": "<redacted>",
          "lastUpdateTime": "2020-07-15T04:33:19.377000+00:00",
          "name": "Project2",
          "revision": 306,
          "state": "wellFormed",
          "url": "https://dev.azure.com/<redacted>/_apis/projects/<redacted>",
          "visibility": "private"
        },
        ...
        {
          "abbreviation": null,
          "defaultTeamImageUrl": null,
          "description": null,
          "id": "<redacted>",
          "lastUpdateTime": "2020-07-20T15:33:57.950000+00:00",
          "name": "Projectn",
          "revision": 670,
          "state": "wellFormed",
          "url": "https://dev.azure.com/<redacted>/_apis/projects/<redacted>",
          "visibility": "private"
        }
      ]
    }
{% endraw %}
~~~

Using the `--query` parameter, we can return a simple array of strings (each entry is a Team Project name) making parsing much simpler: `az devops project list --org $orgUrl --query "value[].name" -o tsv`:

~~~bash
{% raw %}
    /home/colin/repos/foo [master ≡]> az devops project list --org $orgUrl --query "value[].name" -o tsv
    Project1
    Project2
    ...
    Projectn
{% endraw %}
~~~

## Authentication

To authenticate, you need to run `az devops login` which will ask for a Personal Access Token (PAT) to connect to your Azure DevOps organization. Being prompted is fine when you're working in a console, but if you want to automate `az devops` commands, you're going to want to log in without the prompt. To do this, you can set an environment variable called `AZURE_DEVOPS_EXT_PAT` to the value of your PAT. In my `pwsh` (cross-platform PowerShell) scripts, this didn't seem to work totally, so I ended up piping the PAT to the login command too: `$pat | az devops login`.

## Examples

Now that we've got some basics out the way, let's take a look at a few examples.

### Example 1: Creating a Team Project

This one is pretty straightforward: `az devops project create --org $orgUrl --name MyNewProject`. However, in the script I was creating, I wanted to add an organizational group to the `Project Admins` group. That leads to...

### Example 2: Managing Security Groups

Let's get the descriptor (id) of the newly created Team Project's `Project Administrator` Group:

`az devops security group list -p $projectName --org $orgUrl --query "graphGroups[?contains(principalName,'Project Administrators')].descriptor" -o tsv`

We're using `az devops security group` to list out the groups of a Team Project. We're then using the `contains` JMESPath function to query for the node that has the attribute `principalName` like `Project Administrators` and returning the `descriptor` attribute.

Next we query an org-level group by adding `--scope organization` to the command:

`az devops security group list --org $orgUrl --scope organization --query "graphGroups[?contains(principalName,'Specialists')].descriptor" -o tsv`

Here we query the org-level groups looking for a group containing the word `Specialists` and again return the descriptor.

Finally, we use the `group membership` command to add the `Specialists` group to the `Project Admins` group:

`az devops security group membership add --org $orgUrl --group-id $projAdminGroupDescriptor --member-id $specialistGroupDescriptor`

### Example 3: Determining if a Git Repo Exists

Getting a list of Repo names in a Team Project is easy:

`$repoList = az repos list --org $orgUrl -p $ProjectName --query "[].name" -o tsv`

Next I wanted to determine if a repo with a given name existed. Since I'm in `pwsh` on linux, I initially tried `-contains` but this is a case sensitive search. To make the search case insensitive, I convert the list to an `ObjectCollection` and use `FindIndex`:

~~~bash
{% raw %}
    $repoCollection = [Collections.Generic.List[Object]]$repoList
    if ($repoCollection -and $repoCollection.FindIndex({ $args[0] -eq $RepoName }) -ge 0) {
        Write-Host "Repo already exists"
    } else {
        Write-Host "Cloning repo..."
    }
{% endraw %}
~~~

### Example 4: Creating a Git Repo and Importing an External Repo

Before importing a repo, we have to have a repo to import into. To create an empty Git repo, we can use this command:

`az repos create --name $RepoName -p $ProjectName --org $orgUrl`

Next we want to create an import request. In my case, the source repo was another Azure DevOps organization repo and so authentication was required. The same would be true of any private repo. If you require authentication, then generate an authentication token on the source repo and set the environment variable `AZURE_DEVOPS_EXT_GIT_SOURCE_PASSWORD_OR_PAT` to the value of the token (or password). Then the import request will include authentication:

`az repos import create --git-url $sourceRepoURL -p $ProjectName --org $orgUrl --repository $RepoName --requires-authorization`

This creates the request and performs the import from the external Git repo.

### (Bonus) Example 5: Automating Git Commands After Cloning

In our case, we needed to manipulate some files after the import. Assuming we already have a PAT for the new target Team Project (same one we used to authenticate using `az devops login`), we can configure Git to allow `git` operations using `-c http.extraHeader`.

Let's first get the URL of the new repo using:

`$repoUrl = az repos show -p $ProjectName --org $orgUrl -r $RepoName --query "webUrl" -o tsv`

Next, we encode the header to authenticate to the new repo:

`$b64Pat = [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes(":$PAT"))`

Finally, we use the encoded PAT when performing remote `git` operations (like `clone`, `pull` and `push`):

`git -c http.extraHeader="Authorization: Basic $b64Pat" clone $repoUrl`

### Example 6: Deleting a Repo

When you create a new Team Project (using Git) you get a repo with the same name as the Team Project. We wanted to delete that repo. First, get the repo id and then delete it:

~~~bash
{% raw %}
    $repoId = az repos show -r $ProjectName -p $ProjectName --org $orgUrl --query "id" -o tsv
    az repos delete --id $repoId --org $orgUrl -p $ProjectName -y
{% endraw %}
~~~

### Example 7: Creating an ARM Service Endpoint

We needed to create a Service Endpoint to an Azure Subscription in our script. Once again, there is authentication involved because you need an SPN key (we're connecting via SPN and not certificate). Once again, setting an environment variable `AZURE_DEVOPS_EXT_AZURE_RM_SERVICE_PRINCIPAL_KEY` saved the day. Assuming we have the `SPNClientID`, `AzureSubscriptionID` and `Name` and `TenantID`, we can create a service endpoint using:

`az devops service-endpoint azurerm create --azure-rm-service-principal-id $SPNClientID --azure-rm-subscription-id $AzureSubscriptionID --azure-rm-subscription-name $AzureSubscriptionName --azure-rm-tenant-id $TenantID --name $ServiceEndpointName -p $ProjectName --org $orgUrl`

One more thing: once we created the endpoint, we wanted it to be authorized for all pipelines. Fortunately we can do this using `az devops` too! First we retrieve the ID of the newly created endpoint, and then update it:

~~~bash
{% raw %}
    $epId = az devops service-endpoint list --org $orgUrl -p $ProjectName --query "[?name=='$ServiceEndpointName'].id" -o tsv
    az devops service-endpoint update --id $epId --enable-for-all true --org $orgUrl -p $ProjectName
{% endraw %}
~~~

### Example 8: Creating and Deleting (YML) Environments using invoke

Up until this point, I have been able to do everything I needed using "native" `az devops` commands. However, when it comes to manipulating YML environments, there are not yet commands for this in `az devops`. Fortunately, `az devops` provides a "catch all" command called `invoke` that lets you easily invoke any REST API method against Azure DevOps.

Here is the REST API call to list YML environments from this [help doc](https://docs.microsoft.com/en-us/rest/api/azure/devops/distributedtask/environments/list?view=azure-devops-rest-6.0):

`GET https://dev.azure.com/{organization}/{project}/_apis/distributedtask/environments?api-version=6.0-preview.1`

This URL has a "routing parameter" `{project}` and then the portion after the `_apis` specifies the `area` (distributedtask) and `resource` (environments). Finally, we have to note the `api-version`.

Armed with this, we can formulate the request for the `az devops invoke` command:

`$envs = az devops invoke --area distributedtask --resource environments --route-parameters project=$ProjectName --org $orgUrl --api-version "6.0-preview" -o json | ConvertFrom-Json`

> Note: the api-version value is "6-0-preview" without the .1 from the API documentation

We can then query the environments. If we decide that we want to create an environment, we'll need to do a `POST`. `az devops invoke` requires a JSON file as the body of the `POST`, so we can create the body and dump it to a json file and then pass that in as the `--in-file` argument:

~~~bash
{% raw %}
    $envBody = @{
      name = $env
      description = "My $env environment"
    }
    $infile = "envbody.json"
    Set-Content -Path $infile -Value ($envBody | ConvertTo-Json)
    az devops invoke `
       --area distributedtask --resource environments `
       --route-parameters project=$ProjectName --org $orgUrl `
       --http-method POST --in-file $infile `
       --api-version "6.0-preview"
    rm $infile -f
{% endraw %}
~~~

To delete an environment, find its ID:

`$id = az devops invoke --area distributedtask --resource environments --route-parameters project=$ProjectName --org $orgUrl --api-version "6.0-preview" --query "value[?name=='myEnvToDelete'].id" -o tsv`

Once you have its ID, you can use the `DELETE` `http-method` to invoke the REST API DELETE method:

`az devops invoke --area distributedtask --resource environments --route-parameters project=$ProjectName environmentId=$id --org $orgUrl --http-method DELETE --api-version "6.0-preview"`

### Example 9: Creating Variable Groups

We can use `az pipelines variable-group` commands to manipulate variable groups. Here's how we list all the variable groups in a Team Project:

`az pipelines variable-group list -p $ProjectName --org $orgUrl --query "[].name" -o tsv`

To create a variable group, we use the `create` subcommand. We can also specify key/value pairs using the `--variables` arg:

`az pipelines variable-group create --name $varGroupName -p $ProjectName --org $orgUrl --authorize --variables var1="val1" var2="val2"`

### Example 10: Creating a YML Pipeline

To create a YML pipeline, you'll need a name as well as the repository, branch and path to the YML file within the repo (the path relative to the root path of the repo):

`az pipelines create -p $ProjectName --org $orgUrl --name $PipelineName --description $PipelineDescription --repository $RepoName --repository-type tfsgit --branch master --skip-first-run --yml-path $YmlPath`

> Note: The --skip-first-run tells Azure DevOps not to immediately run the pipeline. If you do not specify this, the pipeline is automatically queued.

If you need UI variables for the pipeline, you can add these using `az pipelines variables create` like this:

`az pipelines variable create -p $ProjectName --org $orgUrl --pipeline-name $PipelineName --name myVar --value "some value" --allow-override $true`

> Note: The --allow-override is the checkbox on the UI that allows users to override the variable value at queue time.

## Conclusion

Azure DevOps has a rich REST API and `az devops` certainly makes interacting with the API much easier. This cli is under constant development, so even if there are APIs that are not yet wrapped (like the YML Environment APIs) you can use `invoke` to interact with those APIs. This makes `az devops` a natural choice for scripting and automating Azure DevOps tasks!

Happy commanding!

