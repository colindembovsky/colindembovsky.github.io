---
layout: post
title: Integrating TFS and Project Server – Two Way Manual Sync
date: '2014-04-10 18:55:00'
tags:
- alm
---

I often do road-shows showing off TFS and VS to customers around South Africa. Usually I’m doing this with Ahmed Salijee, the Developer Platform Specialist (DPS) for Developer Tools in Microsoft South Africa. Ahmed is an amazing speaker (we’ve co-presented regularly) and is great at helping customers at a strategic level – and, as he likes to say, for his sins, he gets to help customers with their licensing queries!

Ahmed and I agree on most aspects of Application Lifecycle Management (ALM) using TFS – but one of the places we disagree on is the integration of TFS and Project Server.

## Philosophy: Why You Shouldn’t Be Using Project Server in the First Place

If you don’t care to wax philosophical about Project Server, then skip this section. However, I think it’s important to step back and think about what you’re getting into if you’re planning on using Project Server for tracking software development projects.

Project Plans are made for environments where “change is bad”. Think about what Project Managers do – they create a plan, perhaps entering in requirements, breaking those down into tasks with estimates that they then farm out to team members. Then they do some curious things: they set milestones and they baseline the project. Let’s examine what milestones and baselining mean.

Milestones are points along the way that Project Managers use to answer a simple question: are we conforming to the plan? Baselines communicate an idea: “What we’ve planned is what we value – we should not deviate from this plan. If we do, then it’ll Be Bad”. Project Plans work well when tasks are exactly predictable and repeatable. Unfortunately, that’s not the case in software development. There are many reasons why this is the case – requirements are language-based, and as such are subject to misinterpretations. Coding is an abstract art – and as such it’s extremely difficult to estimate how long a task will take with any accuracy. Even if you get the code bits right, there are always unforeseen issues in integration of components, deployments, testing and so on. This is why people lie about progress – who wants to deviate from the baseline when the baseline is the Ultimate Good? We’d rather bend the truth about how far we are so that, on paper at least, we look good.

That’s why Agile has come to the form. The basic philosophy of any Agile technique is _embrace the change_. If we know things are going to change, then why not embrace the change? Let’s shorten cycle times so that we can get more rapid feedback – that way we minimize the risk of doing the wrong thing for too long. Let’s focus on measuring what value we deliver to business, rather than tracking _conformance to a plan_. Let’s make it concrete: imagine two teams. Team A sticks to the Project Plan closely, and after months of work deliver a system that is average (statistics show that most waterfall-based projects don’t even reach delivery, so we’re being optimistic about Team A’s delivery). Team B deviates widely from their Project Plan, deliver small components frequently and at the end of the project have exceeded expectations. Which team is “better”? The one that stuck to the Plan or the one that Delivered Value? And yet, if we really care about delivering value, why do we beat on teams to Stick To The Plan?

## TFS and Project Server Integration: Good and Bad

So you’re insisting on integrating with Project Server anyway. Fair enough. Let’s examine the [TFS to Project Server integration](http://msdn.microsoft.com/en-us/library/gg455680.aspx).

The good: avoid double entry. That’s about it. The integration allows Work Items added in TFS to be synced over to Project Server (or vice-versa). The integration simply means you don’t have to add (or update progress) in two places.

Perhaps I could add that once you have tasks in a project plan you can do resource leveling and all the other “voodoo” that Project is capable of, but frankly I think that that’s a waste of effort. Let’s imagine you’ve spent a couple of days doing all the leveling. After 3 days, Bob gets sick and is off for 2 days. So you re-level everything, carefully watching as your project drifts from the baseline. Next thing Joe comes and tells you he forgot that one of the changes he’s making will require an extra 2 days of refactoring. You adjust again, sweating a little as your plan deviates further from the baseline. You call up Frank and tell him he’ll have to work the weekend so that you can catch up. Every time something happens, you’re frantically re-adjusting your project plan. At the end, the actual is so far off the baseline, you wonder why you bothered in the first place.

If you train your teams to self-organize, you can track progress of delivered value (which is a much better thing to measure than “conformance to a plan”), rather than dictate who should be doing what when. As changes come in, you embrace the change and adjust course, smiling because change is now a positive thing – not a shame.

If you’re still insisting on integrating TFS and Project Server, there are some caveats that you’ll need to know about before embarking on the integration:

1. Installing the Integration will modify your process templates. The integration adds a bunch of fields and a Project Server form to your work items.
2. You need to configure which PS Projects can be linked to which TFS Team Projects. A PS Project can only be linked to 1 TFS Team Project. If you have a lot of PS Projects (or a lot of TFS Team Projects, or lots of both) you’re going to end up with a lot of integration admin.
3. You need to configure which work items are synced across – at TFS Team Project level. The integration requires you to tell the connector which Work Items it needs to sync. Again, if you have large amounts of PS or TFS Team Projects, you’re looking at a lot of admin.
4. Updating tasks in TFS does not fill in the Timesheet in Project Server. TFS has no knowledge of _when_ work is done – only that work has been done. That means that if you’re going to want to do billing from Project Server, your team members are going to end up filling in Timesheets in Project Server. Updating Timesheets in Project Server does sync actuals and remaining work for work items though.
5. The connector is notoriously hard to debug. If the connector has errors, it can be really hard to track them down.
6. If you have change approvals enabled on Project Server, a project manager can reject changes made to a plan. Imagine a team member updates a work item, which causes the connector to send the change to Project Server. The project manager then rejects the changes. At this point, the sync engine turns off sync for this work item, and the only way to know is to open the work item and take a look. The history has an entry stating the reject reason, and in order to re-sync this work item going forward, you have to re-enable the sync for this one work item.
7. You cannot assign multiple resources to a Task in the Project Server Project Plan. TFS only allows one resource to be Assigned To a Work Item at any one time – which means that if you’re used to multiple resources on the same Task in Project Server, you’re going to have to split the tasks.

## Two Way Manual Sync

So since the integration is so hard (and fragile), perhaps you can consider this alternative: two way manual sync. [This page](http://msdn.microsoft.com/en-us/library/gg593279.aspx) explains in detail the differences between syncing to MS Project versus syncing to Project Server – what I propose here is a “middle ground” that give you best of both worlds without all the pesky configuration required for the integration extension.

Here are the steps to get going:

- On Project Web Access (PWA), create a new enterprise project.
- Open the Project and press “Build Team” and add the resources that will form part of this project.
- Open MS Project and connect to the new Enterprise Project. Check out the plan for editing.
- On the Team Tab, click “Get Work Items” and select a query for the work items you want to bring into the plan.
<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-RxAf9MlfQZc/U0ZoYu3aGdI/AAAAAAAABQI/FWdajgisrU8/image_thumb%25255B2%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-1U07pdzT8So/U0ZoEDsILRI/AAAAAAAABQA/2wR2fSttZGA/image%25255B4%25255D.png?imgmax=800)<!--kg-card-end: html-->
- (Tip: I normally work in the Iteration Backlog and then hit the “Create Query” button to create the iteration backlog query)
<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-v6-6p7k9vdI/U0Zoy9Gzw0I/AAAAAAAABQY/n3nnHwbAcmo/image_thumb%25255B4%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-kYUvjCANm-Q/U0ZogSgS4wI/AAAAAAAABQQ/ztg49Jb4-J4/s1600-h/image%25255B8%25255D.png)<!--kg-card-end: html-->
- Select the work items from the query and click Add.
- Now you can work with the Tasks in Project – leveling resources etc. etc. You can also set a baseline if you want to.
- (Tip: Establish predecessor relationships. Then select all the rows by clicking the row ID – the leftmost column – &nbsp;of the 1st task and then shift-clicking the ID of the last task. Then right click and select “Auto Schedule”. This creates the initial Gantt for you).
<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-RdTNXoOuiMM/U0ZpawIdX_I/AAAAAAAABQo/tF5-8bA1k0M/image_thumb%25255B6%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-Xq6q9B_jy1g/U0ZpVrK1sxI/AAAAAAAABQg/mCMekBn4vgE/s1600-h/image%25255B12%25255D.png)<!--kg-card-end: html-->
- Once you’re done, hit Publish in the Team tab to save your changes back to TFS. In this screenshot, I added a new Task at the bottom of the Project Plan and hit Publish. This then brought back the TFS Work Item id (48).
<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-BHD9yxoxJM4/U0ZpyPl3BdI/AAAAAAAABQ4/wx1vkMzy1QA/image_thumb%25255B8%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-cNpEfJv1BeY/U0ZpprvKNPI/AAAAAAAABQw/Bb_IuqGfppY/s1600-h/image%25255B16%25255D.png)<!--kg-card-end: html-->
- Now you need to publish the changes to Project Server. Hit File and then click the Publish button.
<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-71hMv3PfjjU/U0Zqce9dqjI/AAAAAAAABRI/konInnYntdc/image_thumb%25255B10%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-YFplXjsmygM/U0ZqbY8PEvI/AAAAAAAABRA/1KXsqReAjKQ/s1600-h/image%25255B20%25255D.png)<!--kg-card-end: html-->
- This can take a few seconds, so make sure you watch the status bar to see that the publish succeeded.
<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-tH3HO9xMQJk/U0ZqeM_xRDI/AAAAAAAABRY/Qeh4-xVGssY/image_thumb%25255B14%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-snsqKnV-rbU/U0ZqdQQCvBI/AAAAAAAABRQ/WmymmFuD6sY/s1600-h/image%25255B26%25255D.png)<!--kg-card-end: html-->
- When you close the project plan, make sure you check it in.

Now imagine that the TFS team is updating their tasks. To pull those updates in, let’s open the Project Plan again:

- Go to the Team Tab and press “Refresh”. (Notice in this screenshot that the task actuals / remaining have been updated).
<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-qgqXZ_rsFjE/U0ZqgB8694I/AAAAAAAABRo/1Fyu40l_p50/image_thumb%25255B16%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-2HjkNrHVrJI/U0ZqfNPxn4I/AAAAAAAABRg/Vas5fjPcAdA/s1600-h/image%25255B30%25255D.png)<!--kg-card-end: html-->
- One gotcha: If new tasks were added in TFS, you’ll have to schedule them to see the time estimates (see Task 49 in the above screenshot). It’s a good idea to always hit “auto-schedule” on new (or all) tasks to get new tasks into the Gantt correctly.
- Now you need to publish to Project Server. Again, go to File and click the Publish button.

Finally, consider when new Tasks are added to the Plan in Project Server by another user.

- Open the Project Plan from Project Server
- You’ll immediately see the new tasks
- Add the Work Item Type column and map the new Tasks to work item types. You can also bring in the Area Path and Iteration Path columns.
- Now go to the Team Tab and hit Publish.

## Other Useful Stuff

I wrote a series of posts about integrating TFS and Project server – you can find the fist post [here](http://www.colinsalmcorner.com/2011/07/tfs-and-project-server-integration-tips.html). Also, you can customize the field mappings between TFS and MS Project (not Server) using the guide in [this post](http://www.colinsalmcorner.com/2013/07/adding-custom-team-field-to-ms-project.html). Also, if you’re looking for a Timesheet solution and don’t want to use Project Server, then look out for [Imaginet’s soon-to-be-released new Timesheet product](http://blog.imaginet.com/coming-early-this-summer-imaginet-timesheet-for-tfs-2013-and-visual-studio-online/).

I also can’t recommend highly enough Donald G. Reinertsen’s [The Principles of Product Development Flow](http://www.amazon.com/Principles-Product-Development-Flow-Generation-ebook/dp/B007TKU0O0/ref=sr_1_1?ie=UTF8&qid=1397122619&sr=8-1) – it’ll revolutionize the way you think about delivering value to business.

## Conclusion

Philosophically I think software development teams should stay away from Project Server (or Project Plans) entirely – focus more effort on measuring delivered value to business than conformance to a plan. However, this change is cultural (and needs to be pervasive) so I know that there are teams that are still going to have to integrate to Project Server. Before you embark on the long and painful process of using the server integration, consider using my two way manual sync to see how it works.

Happy Project Tracking!

