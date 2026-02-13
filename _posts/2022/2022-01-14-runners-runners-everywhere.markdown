---
layout: post
title: 'Runners, Runners - Everywhere!'
date: '2022-01-14 01:22:01'
image: /assets/images/2022/01/runners/runner.jpg
description: >
  Hosted runners for Actions are great - but there are some scenarios where you'll need self-hosted runners, such as deploying to private networks. But how can you effectively manage your self-hosted runners? In this post I'll cover some thoughts.
tags:
- build
- deployment
---

1. TOC
{:toc}

> Image from starline on [www.freepik.com](https://www.freepik.com/vectors/background)

GitHub Actions can run on [hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners), which is great because you don't have to maintain infrastructure or keep images up to date. However, you only get a certain amount of free minutes for hosted runners and then you start paying for build minutes. Besides the consumption costs, you may not be able to do certain things with hosted runners, like deploy to private networks.

If you decide to host [self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners), you'll have to decide how to manage them. Registering one or two is fine - but how do you manage dozens or hundreds of self-hosted runners?

One more thing to bear in mind: self-hosted runners are free. That is, you can run as many as you like as often as you like and you don't pay extra to GitHub for them. However, you will need to supply your own compute/storage etc., so that's where the cost for self-hosted runners lies.

# Team Autonomy vs Enterprise Alignment

I use this concept a lot when working with customers after I heard this phrase from the Azure DevOps engineering team (I think it was Aaron Bjork who I first heard it from). The analogy is picturing a container ship vs 600 rowboats. The container ship takes a long time to turn, but you only have to turn one thing. Each of the 600 rowboats can turn nimbly by itself, but getting all 600 pointed in the same direction is a challenge!

What does this have to do with runners? Let's consider two extremes: _centrally managed_ runners vs _team managed_ runners. There are a number of considerations for who manages your runners:

* Who is going to register the runners to GitHub? At what level (more on this later)?
* Who is going to scale up (or down) the runners to optimize cost?
* Who is going to patch runners and keep them up to date?
* Who is going to manage networking for the runners?

## Centrally Managed Runners

Centrally managed runners give you more control. You can define a manageable set of runners that you can share out to teams. This way your infrastructure cost is predictable. However, you'll have to think about capacity - how do you scale up when things get busy? Do you just spin up the maximum number of runners you think you'll need and leave them up 24/7? Or do you want to scale them up/down to lower infrastructure costs?

Centrally managed runners also have the advantage of easier secret management. For example, let's say you want to create a couple of runners that can deploy to the PROD environment. In order to do that, you'll need access to the PROD network - something not all your teams may have. At least with centrally managed runners, you only need to grant access to the centralized team to create the runners on the PROD network. Furthermore, patching runner images is far simpler since they are managed from a centralized team.

Of course, when teams need runners they're going to have to have some mechanism to request them from the central managing team. And what if the runners fall over or need patching? The centralized team is going to have to have some operational headroom to manage the runners.

Futhermore, central teams are going to have to manage access so that one team doesn't snack all the runner capacity from other teams! How runners are divvied up is going to be something the centralized team considers carefully.

## Team Managed Runners

In contrast, team managed runners would leave spinning up, scaling up/down, patching and joining runners to networks totally up to the teams. Teams would have to pay for their own runner infrastructure (somehow) and would need access to target environments (how would they get a runner in PROD, for example?). This has the advantage that teams that require many runners and high capacity don't interfere with teams that require less resources. However, how can the enterprise ensure that teams are patching their runner images?

# Runner Groups

Runners should be organized into [runner groups](https://docs.github.com/en/actions/hosting-your-own-runners/managing-access-to-self-hosted-runners-using-groups), irrespective of how you manage them. The runner groups can live at three levels within GitHub: Enterprise, Organization and Repo.

When you create an Enterprise runner group, you can then specify which organizations within the Enterprise have access to the Enterprise group. Similarly, at the Org level, you can specify which repos have access to the Org groups.

# Permissions

When considering runners, you'll need to think about permissions at a couple levels:

* Permission to register runners with GitHub
* Permissions to target networks

Whoever is registering your runners needs the correct permissions within GitHub: 

* Enterprise runners: `admin:enterprise`
* Organization runners: `admin:org`
* Repo runner: `repo`

Obviously this is going to affect how you manage your runners and whether you lean toward centrally managed or team managed.

Besides permissions in GitHub, the compute for your runner needs access to target environments. If you're creating a runner that can deploy to the PROD network, then whoever is creating the runner infrastructure needs permissions to the PROD network!

# Autoscaling Runners

[Autoscaling self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/autoscaling-with-self-hosted-runners) is not trivial. The only "official" options are the Kubernetes controller (for scaling runners in a Kubernetes cluster) or the AWS Terraform auto-scaler. I have blogged about other mechanisms like using [Azure Container Instances for on-demand ephemeral runner scaling]({% post_url 2021-10-26-on-demand-ephemeral-self-hosted-runners %}).

The point for this discussion is who is going to be responsible for scaling (or auto-scaling) your self-hosted runners. The more you centralize, the more crucial scaling becomes.

# Build vs Deploy Jobs

When coaching teams on effective CI/CD, I use the "build once, deploy many times" mantra. Build jobs should be able to run without environment dependencies. Build jobs be able to check out code, restore dependencies, build/package the code and run unit tests. Unit tests should run in memory and mock out dependencies so that you don't need databases or external systems to complete. If you do that, then your build jobs can run on hosted runners!

Deployment can be a bit trickier. If you're deploying to SaaS services like Azure Web Apps, then you can still use hosted runners and then authenticate using credentials or [using OIDC](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect). However, if you're deploying into a private network or private Kubernetes cluster, then you're going to have to have a self-hosted runner in the target network or cluster.

The point is that you can have a mix of hosted and self-hosted runners in the same workflow! You can target hosted runners for your build jobs and then use self-hosted runners for deploying to private networks. Thus you minimize the number of self-hosted runners you have to manage.

## Spending Limits for Hosted Runners

You can [manage spending limits](https://docs.github.com/en/billing/managing-billing-for-github-actions/managing-your-spending-limit-for-github-actions) for hosted runners at the Enterprise, Org or User level. Presumably most organizations will want to set spending limits at the Org level rather than the user level. Unfortunately, that makes it difficult to limit spending at team level - so if you have several teams in an org, you can't set limits at _team_ level.

# Best of Both

Of course, you don't have to go to either extreme (totally centrally managed or totally team managed). You can have a bit of both. You may have centrally managed groups/runners for PROD environments, but let teams manage their own runners for lower environments and for builds (if they're not able to use hosted agents for builds). This would be my general rule-of-thumb recommendation.

## Premium Runners

Another option on the horizon is [premium runners](https://github.com/github/roadmap/issues/161) which essentially gives you private hosted runners. This is going to be a great option for customers that want the low maintenance of hosted runners but want to be able to allow them to access private networks. Some of the same management considerations (centralized vs team managed) are still going to apply to premium agents though!

# Conclusion

Self hosted runners are essential for most "real world" deployment scenarios. How you manage runners and groups and optimize for availability, operational overhead and cost is going to depend on your culture and how your organization sets up permissions and billing. Some combination of centrally managed and team managed is a good rule of thumb, and fortunately GitHub runners allow you to set the needle wherever you need to.

Happy building!