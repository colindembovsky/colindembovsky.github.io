---
layout: post
title: 'Testing in Production: Routing Traffic During a Release'
date: '2017-05-10 07:13:27'
tags:
- releasemanagement
- devops
---

DevOps is a journey that every team should at least have started by now. Most of the engagements I have been on in the last year or so have been in the build/release automation space. There are still several practices that I think teams must invest in to remain competitive – unit testing during builds and integration testing during releases are crucial foundations for more advanced DevOps, which I’ve blogged about (a lot) before. However, Application Performance Monitoring (APM) is also something that I believe is becoming more and more critical to successful DevOps teams. And one application of monitoring is _hypothesis driven development_.

### Hypothesis Driven Development using App Service Slots

There are some prerequisites for hypothesis driven development: you need to have metrics that you can measure (I highly, highly recommend using Application Insights to gather the metrics) and you have to have a hypothesis that you can quickly test. Testing in production is the best way to do this – but how do you manage that?

If you’re deploying to Azure App Services, then it’s pretty simple: create a deployment slot on the Web App that you can deploy the “experimental” version of your code to and divert a small percentage of traffic from the real prod site to the experimental slot. Then monitor your metrics. If you’re happy, swap the slots, instantly promoting the experiment. If it does not work, then you’ve failed fast – and you can back out.

Sounds easy. But how do you do all of that in an automated pipeline? Well, you can already deploy to a slot using VSTS and you can already swap slots using OOB tasks. What’s missing is the ability to route a percentage of traffic to a slot.

### Route Traffic Task

To quote [Professor Farnsworth](https://en.wikipedia.org/wiki/Professor_Farnsworth), “Good news everyone!” There is now a [VSTS task](https://github.com/colindembovsky/cols-agent-tasks/tree/master/Tasks/RouteTraffic) in my [extension pack](http://bit.ly/cacbuildtasks) that allows you to configure a percentage of traffic to a slot during your release – the Route Traffic task. To use it, just deploy the new version of the site to a slot and then drop in a Route Traffic task to route a percentage of traffic to the staging site. At this point, you can approve or reject the experiment – in both cases, take the traffic percentage down to 0 to the slot (so that 100% traffic goes to the production slot) ad then if the experiment is successful, swap the slots.

### What He Said – In Pictures

To illustrate that, here’s an example. In this release I have DEV and QA environments (details left out for brevity), and then I’ve split prod into Prod-blue, blue-cleanup and Prod-success. There is a post-approval set on Prod-blue. For both success and failure of the experiment, approve the Prod-blue environment. At this stage, blue-cleanup automatically runs, turning the traffic routing to 0 for the experimental slot. Then Prod-success starts, but it has a pre-approval set that you can approve only if the experiment is successful: it swaps the slots.

Here is the entire release in one graphic:

<!--kg-card-begin: html-->[![image](/assets/images/files/269dc0f8-c80b-4a80-8f06-91ca74aeb113.png "image")](/assets/images/files/f863725e-2677-44d2-801b-b47d56b7c930.png)<!--kg-card-end: html-->

In Prod-blue, the incoming build is deployed to the “blue” slot on the web app:

<!--kg-card-begin: html-->[![image](/assets/images/files/900b2a20-072c-4f73-9e35-2a626ca8a461.png "image")](/assets/images/files/3732c09c-f70e-4358-838d-98553f39e19b.png)<!--kg-card-end: html-->

Next, the Route Traffic task routes a percentage of traffic to the blue slot (in this case, 23%):

<!--kg-card-begin: html-->[![image](/assets/images/files/29c131e0-2413-47ec-944c-4c1844dd96e8.png "image")](/assets/images/files/1fdf1cdb-3870-4ea4-a74d-5c38a6d69fb6.png)<!--kg-card-end: html-->

If you now open the App Service in the Azure Portal, click on “Testing in Production” to view the traffic routing:

<!--kg-card-begin: html-->[![image](/assets/images/files/bb224c40-b900-4187-b0ec-8c456be61e8b.png "image")](/assets/images/files/841a61a8-c97c-4be5-948d-876eb20b4d75.png)<!--kg-card-end: html-->

Now it’s time to monitor the two slots to check if the experiment is successful. Once you’ve determined the result, you can approve the Prod-blue environment, which automatically triggers the blue-cleanup environment, which updates the traffic routing to route 0% traffic to the blue slot (effectively removing the traffic route altogether).

<!--kg-card-begin: html-->[![image](/assets/images/files/26843d57-249c-4f73-a686-77af531c6027.png "image")](/assets/images/files/de4dbdb2-e9fc-4772-abc3-3d64f629d583.png)<!--kg-card-end: html-->

Then the Prod-success environment is triggered with a manual pre-deployment configured – reject to end the experiment (if it failed) or approve to execute the swap slot task to make the experimental site production.

<!--kg-card-begin: html-->[![image](/assets/images/files/d79818cf-6180-44ea-8e95-e1939d4f4655.png "image")](/assets/images/files/b68db4d3-1553-4a7f-a7b2-a84ac73f5231.png)<!--kg-card-end: html-->

Whew! We were able to automate an experiment fairly easily using the Route Traffic task!

## Conclusion

Using my new Route Traffic task, you can easily configure traffic routing into your pipeline to conduct true A/B testing. Happy hypothesizing!

