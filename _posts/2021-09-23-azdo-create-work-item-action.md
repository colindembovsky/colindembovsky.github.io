---
layout: post
title: Create Azure DevOps Work Item Action
date: '2021-09-01 17:22:01'
image: /assets/images/posts/azdo.png
description: >
  If you're managing backlogs in Azure Boards but using GitHub Actions for CI/CD, you may have scenarios where you want to create Work Items from an Action.
tags:
- actions
- devops
---

1. TOC
{:toc}

Azure Boards is a mature product for managing backlogs. Many teams are using Boards even while hosting code in GitHub, and using GitHub Actions for CI/CD.

There may be scenarios where you want to create a work item on Azure Boards from within an Action:

- when a Pull Request is created
- when particular tests fail
- when a deployment is commencing or completing

Before we look at one of these scenarios, let's have a look at how I created an Action that can create an Azure DevOps Work Item.

## Azure DevOps Work Item Creation Action

> **tldr;** just go to the Action in the [markeplace](https://github.com/marketplace/actions/create-azdo-work-item) to start using it in your workflows!

### Setting up a Skeleton

I really prefer developing Actions with TypeScript, so I found the [TypeScript Action template repo](https://github.com/actions/typescript-action) and created a repo from the template. This set up a skeleton Action that I could work from. I then opened the Action in a [CodeSpace](https://github.com/features/codespaces) and I was developing! I switched from `npm` to `yarn` but that's not something you have to do.

### Coding the Action

Next I added the [azure-devops-node-api](https://www.npmjs.com/package/azure-devops-node-api) so that I could easily interact with the Azure DevOps REST API. In the [main.ts](https://github.com/colindembovsky/az-create-work-item/blob/main/src/main.ts) file, I use `core.getInput()` to parse the inputs (which I specify in the [action.yml](https://github.com/colindembovsky/az-create-work-item/blob/main/action.yml) file). I then just call the `createWorkItem()` method in a separate [work-item-functions.ts](https://github.com/colindembovsky/az-create-work-item/blob/main/src/work-item-functions.ts) file.

### Authentication

The client library makes authenticating fairly easy, assuming you have an org name and a Personal Access Token (PAT). Here's the method I created to authenticate and get a `WorkItemTrackingClient`, which has methods for interacting with work items:

~~~ts
// file: 'work-item-functions.ts'
async function getWiClient(
  token: string,
  orgName: string
): Promise<IWorkItemTrackingApi> {
  const orgUrl = `https://dev.azure.com/${orgName}`;
  core.debug(`Connecting to ${orgUrl}`);
  const authHandler = azdev.getPersonalAccessTokenHandler(token);
  const connection = new azdev.WebApi(orgUrl, authHandler);

  core.info(`Connected successfully to ${orgUrl}!`);
  return connection.getWorkItemTrackingApi();
}
~~~

Authenticating with Azure DevOps
{:.figcaption}

Notes:
1. We first instantiate an `AuthenticationHandler`, in this case, a PAT handler, passing in the token
1. Then we instanticate the connection, passing in the org URL and the handler
1. Finally, we get and return a `WorkItemTrackingClient`

### Creating a Work Item

Once we have a `WorkItemTrackingClient`, we can manipulate work items in Azure DevOps. Let's look at the method that creates a work item:

~~~ts
{% raw %}
// file: 'work-item-functions.ts'
export async function createWorkItem(
  token: string,
  orgName: string,
  project: string,
  workItemInfo: IWorkItemInfo
): Promise<number> {
  core.debug('Try get work item client');
  const wiClient = await getWiClient(token, orgName);
  core.debug('Got work item client');

  const patchDoc = [
    {op: 'add', path: '/fields/System.Title', value: workItemInfo.title},
    {
      op: 'add',
      path: '/fields/System.Description',
      value: workItemInfo.description
    }
  ] as JsonPatchDocument;

  if (workItemInfo.areaPath !== '') {
    (patchDoc as any[]).push({
      op: 'add',
      path: '/fields/System.AreaPath',
      value: workItemInfo.areaPath
    });
  }
  if (workItemInfo.iterationPath !== '') {
    (patchDoc as any[]).push({
      op: 'add',
      path: '/fields/System.IterationPath',
      value: workItemInfo.iterationPath
    });
  }

  core.debug('Calling create work item...');
  const workItem = await wiClient.createWorkItem(
    null,
    patchDoc,
    project,
    workItemInfo.type
  );

  if (workItem?.id === undefined) {
    throw new Error('Work item was not created');
  }

  return workItem.id;
}
{% endraw %}
~~~

Creating an Azure DevOps Work Item
{:.figcaption}

Notes:
1. First we get the `WorkItemTrackingClient` from the method we created earlier
1. Next, we construct a `JsonPatchDocument` which contains `operations` - in this case, all `add`s. This is how we create the field values for the new work item.
1. Finally, we call `createWorkItem` on the `WorkItemTrackingClient` to create the work item. We then return the `id` of the new work item.

### Tying It All Together
When invoked, the Action will execute the `main.ts` file, in which we just extract the input args and invoke the methods we created earlier:

~~~ts
{% raw %}
// file: 'main.ts'
import * as core from '@actions/core';
import {createWorkItem} from './work-item-functions';

async function run(): Promise<void> {
  try {
    const token: string = core.getInput('token');
    const orgName: string = core.getInput('orgName');
    const project: string = core.getInput('project');
    const type: string = core.getInput('type');
    const title: string = core.getInput('title');
    const description: string = core.getInput('description');
    const areaPath: string = core.getInput('areaPath');
    const iterationPath: string = core.getInput('iterationPath');

    core.debug(`orgName: ${orgName}`);
    core.debug(`project: ${project}`);
    core.debug(`type: ${type}`);
    core.debug(`title: ${title}`);
    core.debug(`description: ${description}`);
    core.debug(`areaPath: ${areaPath}`);
    core.debug(`iterationPath: ${iterationPath}`);

    core.debug('Creating new work item...');
    const newId = await createWorkItem(token, orgName, project, {
      type,
      title,
      description,
      areaPath,
      iterationPath
    });
    core.info(`Created work item [${title}] with id ${newId}`);

    core.setOutput('workItemId', newId);
  } catch (error) {
    core.setFailed((error as any)?.message);
  }
}

run();
{% endraw %}
~~~

The main code for the Action.
{:.figcaption}

### Publishing the Action
I said that the Action invokes `main.ts` - this isn't entirely correct. If you look at the [action.yml](https://github.com/colindembovsky/az-create-work-item/blob/main/action.yml) file, you'll see the following metadata:

~~~yml
{% raw %}
# file: 'action.yml'
runs:
  using: 'node12'
  main: 'dist/index.js'
{% endraw %}
~~~

Metadata snippet for specifying Action exection.
{:.figcaption}

You'll see that the `main` property is set to `dist/index.js`. This file is generated by packaging the Action. This is set up for you already if you create the Action from the template repo like I did. Looking in [pacakge.json](https://github.com/colindembovsky/az-create-work-item/blob/main/package.json) in the `scripts` section, we see the following:

~~~json
"scripts": {
  "build": "tsc",
  "format": "prettier --write **/*.ts",
  "format-check": "prettier --check **/*.ts",
  "lint": "eslint src/**/*.ts",
  "package": "ncc build --source-map --license licenses.txt",
  "all": "yarn run build && yarn run format && yarn run lint && yarn run package"
},
~~~

Scripts specified in the `package.json` file
{:.figcaption}

Note the `all` script: it runs the `build` script for transpiling, then runs `format` and `lint` for formatting and linting and finally runs the `package` command. The `package` command invokes `ncc` to package the Action and make it ready for publication. After these commands have run, the TypeScript has been transpiled, formatted and linted into the `dist` folder.

> Note: For now, I'm running this manually before committing, but ideally I should have an Action that will run this command on `push` and commit the generated `js` files in the `dist` folder automatically - that way I'll never be out of sync!

## Creating a PR Boards Work Item

Let's imagine that you want to create a work item whenever a PR is created. This is actually quite easy now that we have the `az-create-work-item` Action:

~~~yml
{% raw %}
# file: 'pr.yml'
on:
  pull_request:
    types: [opened]

jobs:
  create-work-item:
    runs-on: ubuntu-latest

    steps:
    - run: |
        prNum=$(jq --raw-output .pull_request.number "$GITHUB_EVENT_PATH")
        echo "::set-env name=prNum::$prNum"
      displayName: Exract PR number
    - uses: colindembovsky/az-create-work-item@v1.0.0
      with:
        token: ${{ secrets.AZDO_TOKEN }}
        orgName: myorg
        project: myproject
        type: User Story
        title: PR ${{ env.prNum }} in repository ${{ github.repository }}
        description: '<div>A Pull Request was created <a href="https://github.com/${{ env.repository }}/pulls/${{ env.prNum }}">here</a>.</div>

{% endraw %}
~~~

An example of creating a Work Item in AzDO when a PR is created.
{:.figcaption}

Notes:
1. You can specify "sub-events" for the `pull_request` trigger. In this case, we filter down to the `opened` type.
1. We execute two steps: the first extracts the PR number from the `GITHUB_EVENT_PATH` metadata.
1. The second step invokes the `az-create-work-item` Action to create the work item.
1. For this to work, we need to provide an Azure DevOps PAT and we store it as a repository secret called `AZDO_TOKEN`.

# Conclusion
Creating Azure DevOps work items by using the `az-create-work-item` Action is fairly simple. Many teams are using both GitHub and Azure DevOps, and commonly this involves planning and tracking in Azure Boards. This Action allows teams to easily create work items in Azure Boards from Actions.

Happy building!