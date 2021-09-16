---
layout: post
title: 'Upgrading MSF Agile 5.0 to MSF Agile 6.0: Why does my velocity chart not work?'
date: '2012-08-28 16:42:00'
tags:
- alm
---

So you’ve just upgraded your TFS 2010 server to TFS 2012. And you’ve been using the MSF Agile 5.0 process template. When you open the Web Access webpage, you get a message saying that some features need to be enabled, and you click the link and it "upgrades” your process template so that the Backlogs and Boards work in Web Access. All looks good.

Then you’re doing some Product Backlog planning, and you add some User Stories and you notice that your velocity chart doesn’t show any “active” story points. What’s up?

## The Problem: New States

The “problem” here is that there are new states for the User Story and Task work items in the MSF Agile 6 template. The velocity chart shows any User Story that is “In Progress” (blue) or “Completed” (green).

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-tJ7OUGeWXRA/UDx2SWR3sCI/AAAAAAAAAb8/oexeXX0st8E/image_thumb%25255B2%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-k3UA7cyuroc/UDx2RUqE2dI/AAAAAAAAAb0/G-o84orw-2w/s1600-h/image%25255B4%25255D.png)<!--kg-card-end: html-->

In the figure above, Iteration 1 has 4 story points “delivered” and 13 “in progress” while Iteration 2 has 10 story points “in progress”. However, for this chart to work, you need to put your User Story into the “Resolved” state (which makes it go blue) or “Complete” (which makes it go green). And that’s not ideal, since the User Story is still “active” – it’s not yet actually resolved!

TFS 2012 Process Templates now include two new configuration files: an Agile Process Configuration (for configuring what work items and columns appear on the Backlogs) and a Common Process Configuration for configuring mappings from the boards to work items (among other things like categories). When you click the helpful “enable features” link when you first log into Web Access for the project, TFS creates both the Agile and the Common process config files for you (as well as creating ne work item types like Code Review Request and so on). Let’s export the Common config to see what the “upgrade” process does.

Open a “Developer Command Prompt” (this gets installed with VS 2012 and has a bunch of TFS and VS programs put into the path) and type the following command:

<!--kg-card-begin: html--><font face="Courier New">witamdin exportcommonprocessconfig /collection:http://<em>server</em>:8080/tfs/<em>collection</em> /p:<em>Project</em> /f:<em>Project</em>CommonConfig.xml</font><!--kg-card-end: html-->

(where server, collection and project are your sever, your collection and your MSF Agile 5 project)

If you then open the Common Process Configuration file for your “upgraded” MSF Agile 5.0 Template, you’ll see the following mapping:

    <requirementworkitems category="Microsoft.RequirementCategory" plural="Stories"> <states> <state type="Proposed" value="Active"> <state type="InProgress" value="Resolved"> <state type="Complete" value="Closed"> </state></state></state></states></requirementworkitems>

You’ll notice that the “Proposed” board state maps to the “Active” work item state, that the board state “InProgress” maps to “Resolved” and “Complete” maps to “Closed”. For the velocity chart, any stories in “Proposed” don’t show, any that are “InProgress” are blue and any that are “Complete” are green. And there lies the problem – the User Story from MSF Agile 5.0 doesn’t have enough states for this to work nicely. It would make more sense to add a “New” state and update the mapping.

## The Solution: Add States

What you need to do to “fix” this is to update the User Story and Task work item definitions (essentially adding the “New” state for both work item types) and then update the CommonConfig. Here’s the process for doing this:

1. Go to Team Explorer in VS and connect to your TFS server. On the Home page, click “Settings” and then “Process Template Manager” and export the MSF Agile 6 template to your hard drive.
2. If you did not customize the User Story or Task work item types AT ALL, then skip this step. Otherwise, use witadmin exportwitd to export your User Story and Task work items to file. Then “merge” the User Story and Task definition files (so port over any customizations you did to the type definition in MSF Agile 5 to the MSF Agile 6 type definition). Make sure you end up with the New state (at least) for both work item types.
3. Use witadmin importwitd to import the new User Story and Task definition files from step 2.
4. If you have no state customizations, then simply use witadmin importcommonprocessconfig to upload the MSF Agile 6 common config to your project. If you have other states, make sure you’ve mapped them correctly (for both RequirementWorkItems and TaskWorkItems sections) in &nbsp;before you import.

You’re done! Now you can add User Stories and Tasks (both will go into the “New” state). Then you’ll be able to “commit” to User Stories by transitioning them to Active (when they’ll appear in Blue on your velocity chart). And then you’re good to go!

Happy Agile Planning!

