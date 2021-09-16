---
layout: post
title: 'TFS and Project Server Integration: Tips from the Trenches (Part 1)'
date: '2011-07-07 20:09:00'
tags:
- alm
---

## Links to this series:

- [Part 2 – Setup](http://colinsalmcorner.blogspot.com/2011/07/tfs-and-project-server-integration-tips_07.html)
- [Part 3 – Configuration](http://colinsalmcorner.blogspot.com/2011/07/tfs-and-project-server-integration-tips_165.html)
- [Part 4 – Synchronizing Hierarchies from TFS](http://colinsalmcorner.blogspot.com/2011/07/tfs-and-project-server-integration-tips_6118.html)

Development teams often work in conjunction with a Project Management Office (PMO). A common scenario is the PMO creating high level requirements and the Dev team keeping the Project plans up to date manually in order to report progress back to the PMO. This process means duplication – the PMO task and the TFS requirement. The problem is even worse if the PMO is tracking detailed tasks. Another complication is having to update tasks in both systems.

That’s where the TFS Project Server integration comes into play. This integration keeps TFS and [Project Server](http://www.microsoft.com/project/en/us/default.aspx) up to date via 2 way synchronization. You can kick the tires a bit with [this virtual machine and labs](http://blogs.msdn.com/b/chrisfie/archive/2010/07/06/microsoft-project-server-and-team-foundation-server-2010-ctp-virtual-machine.aspx) that demo the capabilities. You can download the integrator from your MSDN subscription downloads.

I recently worked on an integration for a customer – and these posts are going to detail some of the gotchas that I ran across. This post will focus on the some of the limitations of the integration.

## Supported Scenarios

There are 2 scenario’s that are supported by the Integration: High Level Task Roll-up and Detailed Task Breakdown.

In the High Level Task Roll-up, the PMO creates high level tasks (in Project) that map to requirements in TFS. Only the requirements in TFS are sync’d to Project Server.

In the Detailed Task Breakdown, the requirements are created on the Project Server and then broken down into tasks on TFS. The tasks are also sync’d to Project Server.

This table compares the 2 approaches:

**High Level Task Roll-up**  **Detailed Task Breakdown** &nbsp;Only requirements are sync’d Requirements and tasks are sync’d &nbsp;PMO only gets high level progress PMO can do detailed resource planning &nbsp;Best when mapped to Agile Template Best when mapped to CMMI Template

## Limitations

There are 2 limitations that you need to be aware of when doing the integration.

## Time Tracking

Since TFS has no notion of when work was completed (it tracks only the total work completed and remaining work), you can’t use this synchronization to perform time tracking from the TFS side. If you don’t care about that, then you haven’t got a problem. If you care, then your developers will have to track time in the PWA timesheet.

On a related note, though you can assign multiple resources to a task in Project, you can’t on TFS – so make sure your PMO understands this!

## Hierarchy Sync

This is more of an irritation than a limitation. Project Server has some sort of limitation that it requires parent tasks to exist and be sync’d before the child tasks can be sync’d.

Once you’ve connected a Team Project (in TFS) to an Enterprise Project (in Project Server), you’ll need to tell TFS which work items you want to sync to the Project Server. You do this by setting the “Sync to Enterprise” field on your work item to true and selecting the Enterprise Project you want to sync the work item to.

Here’s the gotcha: if you create a hierarchy in TFS, you need to sync level by level. Start by setting the top level items of the hierarchy to sync – then wait until they are sync’d. Then set the next level of items to sync. Wait for them to sync before next level and so on and so on. Once the items are sync’d you can create child items (and set them to sync) without further problems.

## Names and Permissions

It’s best to use Active Directory for the Enterprise Resources. The sync engine matches Enterprise Resources to TFS users using the display name – if you’re not using Active Directory Sync in the PWA, then make sure the display name of the Enterprise Resource matched the Display Name of the AD User exactly.

If your PMO has customized permissions for Enterprise users in the PWA, then you may run into issues. For example, if your PMO does not grant the “Create Tasks” permission (if they don’t want everyone creating tasks in the Project) then the sync engine won’t be able to sync from TFS to Project Server – the engine uses the “Created By” identity to create tasks on the Enterprise Project.

In the next post, I’ll talk about setup and configuration of the sync engine.

