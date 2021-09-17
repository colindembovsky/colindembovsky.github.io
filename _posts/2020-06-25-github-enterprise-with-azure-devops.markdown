---
layout: post
title: 'Hosting Code On Premises: GitHub Enterprise with Azure DevOps'
date: '2020-06-25 19:12:33'
description: >
  Do you want to be on the latest DevOps platforms, but are required to keep source code on premises? In this post I talk about considerations for hosting GitHub Enterprise and Azure DevOps Server on premises.
tags:
- github
- devops
---

1. TOC
{:toc}

I've been working recently with several customers that are migrating source code to GitHub Enterprise. Most of these customers are also on Team Foundation Server (TFS) or Azure DevOps Server (the newer version of TFS).

In this post I want to point out some considerations when migrating to GitHub Enterprise:

1. **CI/CD** : [GitHub Actions](https://github.com/features/actions) are not currently available on GitHub Enterprise, so how do you perform automated build and release?
2. **Work Item Tracking** : GitHub Projects and Issues may not be adequate for work item tracking, especially when the organization requires Portfolio Management
3. **Cloud Hosting vs Hosting On Premises** : Hosting GitHub Enterprise (and Azure DevOps Server) requires infrastructure and introduces operational overhead. Are there other options?

## Why Host Source Code On Premises?

Ideally, customers _should_ host source code in the cloud - on GitHub.com or on Azure DevOps Server. Even if you do not (yet) deploy to the cloud, you can still perform builds in the cloud and have on premises agents on your network to deploy to on premises environments.

However, there are organizations that have regulatory requirements or their InfoSec teams do not (yet) have enough trust in cloud providers. I recommend that teams have an open discussion with InfoSec to find out how hosted cloud platforms like GitHub and Azure DevOps are very secure and meet stringent compliance requirements. Use the [Azure Compliance page](https://azure.microsoft.com/en-us/overview/trusted-cloud/compliance/) as well as the [Azure DevOps Data Protection overview page](https://docs.microsoft.com/en-us/azure/devops/organizations/security/data-protection?view=azure-devops) in your discussions.

Still, many organizations will mandate that source code be kept on premises - so how can you achieve this goal and still make use of cutting edge platforms?

## CI/CD

[GitHub Actions](https://github.com/features/actions) are promising, but are not yet available on GitHub Enterprise Server. Additionally, while Actions are great for CI, there are still a number of features that are not yet implemented in order to support sophisticated CD (including _environments_ and _approvals_, among others). GitHub advanced security is also not yet available - though I expect that GitHub Enterprise Server will eventually catch up to GitHub.com.

[Azure Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/) is available in Azure DevOps Server 2019. However, the same limitations apply in that [environments](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/environments?view=azure-devops) (and therefore approvals) are not yet available (though they will be in Azure DevOps Server 2020 which is imminent).

Assuming that you can wait for Azure DevOps Server 2020, the ideal scenario is that you have GitHub Enterprise for source control and integrate to Azure DevOps Server 2020 for Boards (work item tracking) and Pipelines (using YML files in the repos).

## Work Item Tracking

Azure Boards let you track work according to iterations (or sprints) or just visualize and flow work using Kanban. It has rich customization capabilities and ties neatly into Azure Pipelines and other Azure DevOps capabilities.

If you have any kind of Portfolio (cross-team) management requirements, then using Azure Boards is arguably going to be better than using GitHub projects and issues.

Again the ideal scenario is using GitHub Enterprise Server for source control and Azure Boards for work item tracking.

## Cloud Hosting vs Hosting On Premises

There are several drawbacks to hosting on premises:

1. You need to maintain infrastructure - VMs, SQL Servers, backups, patching - you'll need to have someone perform all of these operations
2. Upgrade cycles - you'll need to upgrade GitHub Enterprise and Azure DevOps Server periodically to get new features, and these upgrades can be disruptive
3. No extranet access - you will have to VPN into your network in order to access your source control, which makes working from home more challenging and makes it harder to utilize contractors

Is there an alternative? There are two. The first is to use [GitHub Private Instances](https://github.blog/2020-05-06-new-from-satellite-2020-github-codespaces-github-discussions-securing-code-in-private-repositories-and-more/#private) (which are not publicly available yet). These are instanced of GitHub Enterprise that are fully managed in Azure, but can be placed on to a private network. This may be good enough for your InfoSec teams.

The second alternative, which is possible right now, requires opening a firewall port to your source control system, which InfoSec may not allow. But if you can convince them to open port 443 to a whitelist of Azure DevOps IPs, then you can use this architecture:

1. Have source code in a local GitHub Enterprise Server
2. Open a port (443) in the corporate firewall to the whitelist describe [here](https://docs.microsoft.com/en-us/azure/devops/organizations/security/allow-list-ip-url?view=azure-devops)
3. Create an Azure DevOps Services (cloud) account
4. Create a Service Endpoint from Azure DevOps Services to your on premises GitHub Enterprise server (this is why the firewall opening is required)
5. Run builds using [hosted agents](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops&tabs=browser#microsoft-hosted-agents) OR
6. Run builds using [private (self-hosted)](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops&tabs=browser#install) agents
7. Deploy on premises using private agents

In this scenario, you only have to maintain infrastructure for GitHub Enterprise, but your Pipelines and Boards are on the cloud.

## Conclusion

Ideally, you want to host your source code in the cloud. If that is not an option, then I recommend using GitHub Enterprise Server on premises and Azure DevOps Services (cloud) for Boards and Pipelines, since this has the least amount of operational overhead and infrastructure requirements and removes the Azure DevOps upgrade cycle. However, this requires opening a firewall port to your GitHub Enterprise Server. If this is not an option, then you're left with having to host both GitHub Enteprise Server and Azure DevOps Server locally. Not ideal, but al least you'll be "near current" for your DevOps platform.

