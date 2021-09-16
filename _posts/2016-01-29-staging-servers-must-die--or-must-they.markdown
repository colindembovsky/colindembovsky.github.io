---
layout: post
title: Staging Servers Must Die – Or Must They?
date: '2016-01-29 17:12:37'
tags:
- devops
---

Edith Harbaugh published a though-provoking post called [Staging Servers Must Die](http://readwrite.com/2016/01/22/staging-servers) with the byline “so continuous delivery may live.” She asserts something which I’d never really considered before: that separate, cascading Dev, QA, Staging and Prod environments is a hangover from Waterfall development.

## Agility = No Build or Staging?

Harbaugh makes some bold assertions about what she calls “DevOps 2.0”. First, she states that teams should ditch the concept of build (which she calls antiquated). Developers should be checking source into their mainline and and deploying immediately to Prod – with feature flags. The flag of a new feature being deployed defaults to “off for everyone” – no need to keep staging in sync with Prod, and no delay. The QA team are then given access to the feature, then beta customers, and slowly the number of users with access to the feature is increased until everyone has it and the feature is “live”.

She calls out four problems with cascading environments. The first one is time: she argues that a pipeline of environments slows delivery since builds have to be queued and then progressively moved through the pipeline. Secondly, staging environments increase costs since they require infrastructure. Thirdly, she says that the effectiveness of staging environments is moot since they can almost never reproduce production exactly. Finally, she recounts bad experiences where she needed users to test on staging servers, and users continually logged into Prod instead of Staging (or vice-versa) and so the effectiveness of having a staging environment became eclipsed user confusion.

## Feature Flags

I think that Harbaugh’s view of feature flags may be a tad biased, since she is the CEO of a [LaunchDarkly](https://launchdarkly.com), a product that allows developers to introduce and manage feature flags. Still, feature flags are a great solution to some of the challenges she lists. However, feature flags are hard to code and manage (that’s why she has a product that helps teams manage it).

LaunchDarkly is a really neat idea – in your code, you call an API that queries LaunchDarkly to determine if this feature is on for this user. Then you can manage which users have which features outside the app in LaunchDarkly – great for A/B testing or releasing features to beta customers and so on.

Feature flags always sound great in theory, but how do you manage database schema differences? How do you fix a bug (what bug?) – do you need a feature flag for the bug fix? What about load testing a new feature – do you do that against Prod?

## Agility

So are feature flags and ditching builds and staged environments the way to increase agility and progress to “DevOps 2.0”? It may be in some cases, but I don’t think so. Automated deployment doesn’t make you DevOps – DevOps is far more that just that.

Here are my thoughts on what you should be thinking about in your DevOps journey.

### Microservices

You may be able to go directly to microservices, but even if you can’t (and in some cases you probably shouldn’t), you should be thinking about breaking large, monolithic applications into smaller, loosely-coupled components. Besides better architecture, isolating components allows you _deployment granularity_. That is, you can deploy a component of your application without having to deploy the entire application. This makes for much faster cycles, since teams that complete functionality in one component can deploy immediately without waiting for teams that are working on other components to be ready to deploy. Smaller, more frequent, asynchronous deployments are far better than large, infrequent, synchronized deployments.

### Automated Builds with Automated Testing

This has always seemed so fundamental to me – I battle to understand why so many dev teams do not have [builds](/automated-buildswhy-theyre-absolutely-essential-(part-1)) and [unit tests](/why-you-absolutely-need-to-unit-test). This is one of my biggest disagreements with Harbaugh – when a developer checks in, the code should trigger a build that not only compiles, but goes through a number of quality checks. The most non-negotiable is unit testing with coverage analysis – that way you have some measure of code quality. Next, consider static code analysis, and better yet, integration with [SonarQube](http://www.sonarqube.org/) or some other technical debt management system.

Every build should produce metrics about the quality of your code – tests passed/failed, coverage percentage, maintainability indexes and so on. You should know these things about your code – deploying directly to production (even with feature switches) bypasses any sort of quality analysis on your code.

Your build should also produce a deployable package – that is _environment agnostic_. You should be able to deploy your application to any environment, and have the deployment process take care of environment specific configuration.

Beyond unit testing, you should be creating automated integration tests. These should be running on an environment (we’ll discuss that shortly) so that you’re getting quality metrics back frequently. These tests typically take longer to run than unit tests, so they should at least be run on a schedule if you don’t want them running on each check-in. Untested code should never be deployed to production – that means you’re going to have to invest into keeping your test suites sharp – treat your test code as “real” code and help it to help you!

### Automated Deployment with Infrastructure As Code

Harbaugh does make a good point – that cascading dev/test/staging type pipelines originate in Waterfall. I constantly try to get developers to separate _branch_ from _environment_ in their minds – it’s unfortunate that we have dev/test/prod branches and dev/test/prod environments – that makes developers think that the code on a branch is the code in the environment. This is almost never the case – I usually recommend a dev/prod branching structure and let the _build_ track which code is in which environment (with proper versioning and labeling of course).

So we should _repurpose_ our cascading environments – call them integration and load or something appropriate if you want to. You need somewhere to run all these shiny tests you’ve invested in. And go Cloud – pay as you use models mean that you don’t have to have hardware idling – you’ll get much more efficient usage of environments that are spun up/down as you need them. However, if you’re spinning environments up and down, you’ll need to use Infrastructure as Code in some form to automate the deployment and configuration of your infrastructure – [ARM Templates](https://azure.microsoft.com/en-us/documentation/templates/), [DSC scripts](https://msdn.microsoft.com/en-us/powershell/dsc/overview) and the like.

You’ll then also need a tool for managing deployments in the pipeline – for example, [Release Management](https://msdn.microsoft.com/en-us/library/vs/alm/release/overview-rmpreview). Release Management allows you to define tasks – that can deploy build outputs or run tests or do whatever you want to – in a series of environments. You can automate the entire pipeline (stopping when tests fail) or insert manual approval points. You can then configure triggers, so when a new build is available the pipeline automatically triggers for example. Whichever tool you use though, you’ll need a way to monitor what builds are where in which pipelines. And you can of course deploy directly to Production when it is appropriate to do so, so the pipeline won’t slow critical bugfixes if you don’t want it to.

### Load Testing

So what about load and scale testing? Doing this via feature switches is almost impossible if you don’t want to bring your production environment to a grinding halt. If you’re frequently doing this, then consider replication of your databases so that you always have an exact copy of production that you can load test against. Of course, most teams can use a subset of prod and extrapolate results – so you’ll have to decide if matching production _exactly_ is actually necessary for load testing.

Having large enough datasets should suffice – load testing should ideally be a relative operation. In other words, you’re not testing for an absolute number, like how many requests per second your site can handle. Rather, you should be base lining and comparing runs. Execute load test on current code to set a base line, then implement some performance improvement, then re-run the tests. You now compare the two runs to see if your tweaks were effective. This way, you don’t necessarily need an exact copy of production data or environments – you just need to run the tests with the same data and environment so that comparisons make sense.

### A/B Testing

Of course feature switches can be manipulated and managed in such as way as to enable A/B testing – having some users go to “version A” and some to “version B” of your application. It’s still possible to do A/B testing without deploying to production – for example, using [deployment slots](https://azure.microsoft.com/en-us/documentation/articles/web-sites-staged-publishing/) in Azure. In an Azure site, you’d create a staging slot on your production site. The staging slot can have the same config as your production slot or have different config, so it could point to production databases if necessary. Then you’d [use Traffic Manager to divert some percentage of traffic to the staging slot](https://azure.microsoft.com/en-us/documentation/articles/app-service-web-test-in-production-get-start/) until you’re happy (which the users will be unaware of – they go to the production URL and are none the wiser that they’ve been redirected to the staging slot). Then just swap the slots – instant deployment, no data loss, no confusion.

## Conclusion

Staging environments shouldn’t die – they should be repurposed, rising like a Phoenix out of the ashes of Waterfall’s cremation. Automated builds with solid automated testing (which requires staging infrastructure) should be what you’re aiming for. That way, you can deploy into production quickly _with confidence,_ something that’s hard to do if you deploy directly to production, even with feature switches.

Happy staging!

