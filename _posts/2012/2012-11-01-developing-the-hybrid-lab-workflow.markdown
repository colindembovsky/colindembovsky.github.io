---
layout: post
title: Developing the Hybrid Lab Workflow
date: '2012-11-01 19:12:00'
tags:
- build
- labmanagement
---

In my [previous post](http://colinsalmcorner.blogspot.com/2012/10/hybrid-lab-workflow-standard-lab.html) I talked about my [Hybrid Lab Workflow](http://hybridlabworkflow.codeplex.com/) – this workflow allows you to do a Build-Deploy-Test workflow against a TFS 2012 Standard Environment, and as long as the environment is composed of VMs and you’re able to connect to the VM Host, then you can apply pre-deployment snapshots and take post-deployment snapshots.

## Why the Wizard needs the Interactive Agent Password

The trickiest bit of this workflow was getting the Environment ready for Deployment and Testing after a snapshot was applied. I discovered that if you took a snapshot of each the VMs in the Lab after you created the Environment, then if you applied the snapshot the Lab would look ready, but when you tried to Deploy into the Lab (specifically the machine that is running it’s lab agent as an interactive process for Coded UI tests) the deployment task failed.

I discovered (by chance) that if you restart the Interactive Agent on the test machine, the Deployment and Test worked as expected.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-QxGCUWGq5Sw/UJJK19OZXwI/AAAAAAAAAgs/QLK230DnMJc/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-MovEP7xbgkg/UJJK0jUXV9I/AAAAAAAAAgk/WiFnrVszGMc/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

The problem was then achieving this in the workflow, since there is no API for doing this. I tried to achieve this from the controller side too (since if you “offline” and then “online” the agent from the controller in MTM, the lab worked too). Again, no luck, since there’s no API.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-avWuWrMO40s/UJJK5VxjfwI/AAAAAAAAAg8/5jA9XarzRNI/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-xOEhFGtrtQU/UJJK31gfsUI/AAAAAAAAAg0/O4AIO27f9Uo/s1600-h/image%25255B7%25255D.png)<!--kg-card-end: html-->

So that’s why I had to add some nastiness to the Wizard and to the workflow. In the Wizard when you create a workflow, you need to provide the password for the Test Agent user – I can get the username from the Lab API.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-yQMwvqHHcxs/UJJK8f_q6-I/AAAAAAAAAhM/4i5fDqDHPXY/image_thumb%25255B5%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-j5ncgGxv6uI/UJJK7CvGPHI/AAAAAAAAAhE/LIHFCHogNIA/s1600-h/image%25255B11%25255D.png)<!--kg-card-end: html-->

Once I have that, I can use PsKill to remotely stop the Test Agent and then PsExec to remotely start the Agent again – that wires it up nicely and the rest of the workflow continues normally.

There was another alternative – I could invoke the LabEnvironment.Repair() method and specify that the agent needs to be installed on the test machine. This worked, but it took about 15 minutes for the Lab infrastructure to figure out that the Agent was already installed and then to wire it up. Resetting the Agent remotely only takes about 30 seconds, so I decided to go with that instead. I would have preferred a way to do this without a password or PsKill and PsExec, but since there’s no API for this, I went with the “quicker, dirtier” method.

Anyway, if you really don’t like that you can leave the password blank. The bit of the workflow that uses PsKill and PsExec will be bypassed – but then your Agent may be in some weird limbo state because of the snapshot. Maybe this only happens in my scenario and you don’t need to do it.

Happy workflowing!

