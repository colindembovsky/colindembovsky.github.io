---
layout: post
title: Enterprise GitHub
date: '2019-08-14 06:54:11'
tags:
- sourcecontrol
---

1. TOC
{:toc}

Since [Microsoft acquired GitHub](https://news.microsoft.com/2018/06/04/microsoft-to-acquire-github-for-7-5-billion/), and the anti-Microsoft folks had calmed down, there have been a number of interesting developments in the GitHub ecosystem. If you’ve ever read one of my blogs or attended any events that I’ve spoken at, you’ll know that I am a raving Azure DevOps fan. I do, however, also have several [repos on GitHub](https://github.com/colindembovsky). As a DevOpsologist (someone who is in a constant state of learning about DevOps) I haven’t ever recommended GitHub for Enterprises – but the lines of functionality are starting to blur between GitHub and Azure DevOps. So which should you use, and when?

One note before we launch in: Azure DevOps is the name of the suite of functionality for the Microsoft DevOps platform. There are a number of “verticals” within the suite, which you can mix and match according to your needs. [Azure Repos](https://azure.microsoft.com/en-us/services/devops/repos/) is the source control feature, [Azure Boards](https://azure.microsoft.com/en-us/services/devops/boards/) the project management feature, [Azure Test Plans](https://azure.microsoft.com/en-us/services/devops/test-plans/) the manual test management feature, [Azure Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/) the Continuous Integration/Continuous Deployment (CI/CD) feature and [Azure Artifacts](https://azure.microsoft.com/en-us/services/devops/artifacts/), the package management feature.

## Blurring the Line

Up until a couple years ago, the line between GitHub and Azure DevOps was fairly simple to me: if you’re doing open-source development, use GitHub. If you’re doing Enterprise development, use Azure DevOps. The primary reasons were that GitHub was mainly just source control with some basic issue tracking, and you got unlimited _public_ repos (at least, private repos were a lot more expensive). Azure DevOps (or Visual Studio Team Services, VSTS, as it used to be called), on the other hand, offered Project Management, CI/CD, Test Management and Package Management in addition to unlimited _private_ repos. However, now you can now create private repos inexpensively in GitHub and you can create public repos in Azure DevOps. And last week, GitHub announced an update to [GitHub Actions](https://github.com/features/actions) that enables first-class CI, which conceivably will expand to CD fairly soon. While you can’t do manual Test Management using GitHub, you get a far better “InnerSource” experience in GitHub than you do in Azure DevOps.

## This Shouldn’t Be A Surprise

This isn’t entirely apples-to-apples, because (to oversimplify a bit) Azure DevOps is a DevOps platform, while GitHub is primarily a source control platform. At least, that’s been my experience up until recently. GitHub has some good, basic project management capabilities that work fantastically for open source development, but is a little simplistic for Enterprise development. As the industry shifts more and more to automated testing over manual testing, fewer teams have a need to manage manual testing. While you can publish releases on GitHub repos, Azure Artifacts arguably offers a more feature-rich service for package management. Also, while Azure DevOps, when it was still Team Foundation Server (TFS), used to be a “better together” suite, where you were probably better off doing everything in TFS, Azure DevOps is embracing the fact that developers have diverse toolsets, languages, platforms and target platforms. You can now very easily have source code in GitHub, Project Management in Jira, CI/CD in Azure Pipelines and have good visibility and traceability end-to-end.

We shouldn’t be surprised by the blur between GitHub and Azure DevOps. After all, Microsoft owns both now. And I think acquiring GitHub was an astute move by the tech giant – because, _if you win the developer, you’re likely to win the organization_. The perception of a “big bad Microsoft” is rapidly changing. Even before the GitHub acquisition, Microsoft employees were the [top contributors](https://www.techrepublic.com/article/microsoft-may-be-the-worlds-largest-open-source-contributor-but-developers-dont-yet-care/) to open source projects in GitHub. So not only is Microsoft embracing open source more and more, but they purchased the premier open source platform in the world!

The question then becomes: Where is Microsoft focusing? Are you better off in GitHub or in Azure DevOps, or some Frankenstein mix of the two? Will Azure Repos continue to evolve, or will Microsoft “Silverlight” Azure Repos? If I could gaze into a crystal ball, I’d predict that in 3 – 5 years, most organizations will have source code in GitHub and utilize Azure DevOps for Project Management, Pipelines and Package Management. Disclaimer: this is pure conjecture on my part!

## Azure DevOps and GitHub Head to Head

The blurred lines between GitHub and Azure DevOps should be cause for celebration, not consternation. It just means that we have more options! And options are good, if you consider them carefully and don’t make hype-based decisions. So let’s compare Azure DevOps and GitHub head to head in the realms of Source Control, Project Management, CI/CD and Package Management.

### Source Control

There’s not much difference as far as source control management goes between Azure Repos and GitHub, if you’re talking about Git. Fortunately, when the Azure DevOps team decided to add distributed version control to Azure DevOps, they just added Git. Not some funky Microsoft version of Git (though they contribute actively to Git and specifically to [libgit2](https://libgit2.org/), the core Git libraries). So if you have a Git repo, you can add a GitHub remote, or an Azure DevOps remote, or both. Both are just Git remote repositories. However, if you want centralized source control ([don’t do this any more](/modernizing-source-control---migrating-to-git)) then you have to go with Azure DevOps. I would argue that the Pull Request experience is slightly better in Azure DevOps, but not by much. Both platforms allow you to protect branches using policies, so not much difference there either. Both platforms have WebHooks that you can use to trigger custom actions off events. Both have APIs for interaction. GitHub Enterprise has pre-receive hooks that can validate code before it is actually imported into the repo. Azure DevOps has a similar mechanism for centralized version control with [Check-In policies](https://docs.microsoft.com/en-us/azure/devops/repos/tfvc/add-check-policies?view=azure-devops), but these do not work for Git repos. We’ll call this one a tie.

### Project Management

Azure Boards has a better Enterprise Project Management story. With GitHub you get Issues and Pull Requests (PRs) as the base “work item” types. You can add Task Lists to Issues, but the overall forms and flows of Issues and PRs is basic. Azure Boards work items can be a lot more complex, but offer much more customization opportunities. You can also do portfolio management more effectively in Azure Boards, since you can create work item hierarchies. GitHub does have the notion of Milestones and Projects, but again the functionality is fairly basic and probably too simplistic for Enterprises. While you can create basic filters for work items in GitHub, Azure DevOps has an advanced [Work Item Query Language](https://docs.microsoft.com/en-us/azure/devops/boards/queries/wiql-syntax?view=azure-devops) and [elastic search](https://docs.microsoft.com/en-us/azure/devops/project/search/overview?view=azure-devops). Both platforms allow you to tag (or label) work items. Azure Boards also lets you create widgets and dashboards and even has an [OData feed](https://docs.microsoft.com/en-us/azure/devops/report/powerbi/create-quick-report-odataq?view=azure-devops) and an [Analytics Service](https://docs.microsoft.com/en-us/azure/devops/report/powerbi/what-is-analytics?view=azure-devops) so that you can create reports (say from PowerBI) over your work items. Of course you could use neither system for Project Management, you could use Jira, integrating Jira tickets easily to both Azure Repos (and Pipelines) or GitHub.

In terms of Enterprise project management, I’d have to give this one to Azure Boards.

### CI/CD

GitHub introduced GitHub actions about a year ago. Hundreds of Actions were created by the community, validating the demand for actions triggered off events on a repo. But it seemed that doing any sort of Enterprise-scale CI with Actions was a challenge. Last week, a new and improved version of GitHub Actions was announced, and now CI is baked into GitHub through GitHub actions. I expect that we’ll see a huge surge in adoption of this CI tool and platforms like CircleCI and other cloud-CI systems may battle to compete. The feature isn’t GA yet, so we’ll see. It’s also suspiciously close to the YML format used by Azure Pipelines and supports Windows, Mac and Linux, just like Azure Pipelines…

The story doesn’t quite end there – if you want CI for GitHub repos, you now have a choice of GitHub Actions or Azure Pipelines, since Azure Pipelines has native support for GitHub repos. If you have repos outside of GitHub, you can’t use Actions.

I’d have to give this one to Azure Pipelines, at least for now. Azure Pipelines does include Release Management (for CD) or [multi-stage YML](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/stages?view=azure-devops&tabs=yaml) files. I am sure we’ll see similar support soon for Actions, but for now while you can do CI pretty easily in GitHub Actions, CD is going to be a challenge. At this point in time, I’ll give this one to Azure Pipelines.

### Package Management

GitHub releases allow you to tag a repo and publish that version of the repo as a release. You can also upload binaries that are the versioned packages for that release. Azure Artifacts allows you to create feeds that can be consumed – you can access a feed using [NuGet](https://docs.microsoft.com/en-us/azure/devops/artifacts/get-started-nuget?view=azure-devops), [npm](https://docs.microsoft.com/en-us/azure/devops/artifacts/get-started-npm?view=azure-devops&tabs=windows), [Maven](https://docs.microsoft.com/en-us/azure/devops/artifacts/get-started-maven?view=azure-devops), [Python packages](https://docs.microsoft.com/en-us/azure/devops/artifacts/quickstarts/python-packages?view=azure-devops) or [Universal Packages](https://docs.microsoft.com/en-us/azure/devops/artifacts/quickstarts/universal-packages?view=azure-devops&tabs=azuredevops) (which is a feed of arbitrary files – think NuGet for anything). Feeds are usually better than releases since tools like NuGet or npm know how to connect to feeds. Again, in terms of Enterprise package management, this one goes to Azure Artifacts.

## Just Tell Me Which One To Use Already!

So the final score is 3.5 to 0.5 for Azure DevOps. But that’s not a full reflection of the situation, so don’t start porting to Azure DevOps just yet. Remember, options are great if you consider them _carefully_. And they’re not mutually exclusive either. So here is what I think are the key considerations:

### Unit Of Management: Code or Work Items?

Do you track your work by looking at work item tracking, reporting and rolling up across your portfolio? Then you probably need to look at Azure Boards, since GitHub Issues are not going to handle complex Enterprise-level portfolio management. However, if your teams operate a bit more independently and you track work by looking at changes to repos, GitHub may be a better fit. Don’t forget that you can still keep source code in GitHub and use Azure Boards for project management!

### Single Pane of Glass or Bring Your Own Tools?

I’ve seen Enterprises that are trying to standardize tooling and processes across teams. In this case, since Azure DevOps is a complete end to end platform, you’re probably better off using Azure DevOps. If you prefer a smorgasbord of tools, you could go either way. Even if you are managing work with Jira, building with TeamCity and deploying with Octopus Deploy, Azure DevOps could still tie these tools together to serve as a “backbone” giving you a single pane of glass.

### Manual Test Management or Centralized Source Control?

If you need a tool for Manual Test Management, then Azure DevOps is for you. However, you could easily keep source code in GitHub and still use Azure Test Management to manage manual tests. And if you need centralized source control for some reason, then your only option is Azure Repos using Team Foundation Version Control (TFVC).

## Conclusion

As you can see, there’s a lot of overlap between GitHub and Azure DevOps. Here’s my final prediction – we’ll see more innovation in the source control space in GitHub than we will in Azure Repos. Once again, the disclaimer is that this is my observation, and is in no way based on any official communication from either GitHub or Azure DevOps. I do think that a very viable option for Enterprises in the next few years will be to manage source code in GitHub and use Azure DevOps for CI/CD, Project Management and Package Management. This gives you the best of both worlds – you’re on (arguably) the best source control system – GitHub – and you get Enterprise-grade features for project, build and package management. As GitHub Actions evolves, perhaps CI/CD can be moved over to GitHub too. Yes, it’s still fuzzy even after thinking through all these options!

In short, think clearly about which platform (or combination of platforms) is going to best suit your Enterprise’s culture. Remember, DevOps is as much about people (culture) as is is about tooling.

Happy DevOps!

