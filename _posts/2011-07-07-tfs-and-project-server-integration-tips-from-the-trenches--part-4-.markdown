---
layout: post
title: 'TFS and Project Server Integration: Tips from the Trenches (Part 4)'
date: '2011-07-07 20:17:00'
tags:
- alm
---

## Links to this series:

- [Part 1 – Considerations](http://colinsalmcorner.blogspot.com/2011/07/tfs-and-project-server-integration-tips.html)
- [Part 2 – Setup](http://colinsalmcorner.blogspot.com/2011/07/tfs-and-project-server-integration-tips_07.html)
- [Part 3 – Configuration](http://colinsalmcorner.blogspot.com/2011/07/tfs-and-project-server-integration-tips_165.html)

## Part 4: Synchronizing Hierarchies from TFS to Project

One limitation of the synchronization is that you need to sync parent items before child items. One way of getting work items into TFS and Project is to use Excel. Open Excel, go to the Team Tab and click “New List”. In the dialogue, select the Team Project that is associated and select “Input List” to get some default columns. In the ribbon at the top, select “Choose Columns” and add “Project Server Submit” and “Project Server Enterprise Project”. Finally, press the “Add Tree Level” button to add a level for each level in your hierarchy.

Now you can enter your work items.

When you’re ready to submit to TFS, change the Project Server Submit value on **only** the 1st level of the hierarchy to “Yes” and select the Enterprise Project in the Project Server Enterprise Project column (there will be a value here for each Enterprise Project that is associated to this Team Project). In the image below, you can see that I’ve only set the High Level Requirements to sync.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-m4sQKrnaa0w/ThWVmb8gZOI/AAAAAAAAAR4/OF99BnFSmEc/image_thumb5.png?imgmax=800 "image")](http://lh5.ggpht.com/-DiUEKWlOL2M/ThWVlTpOusI/AAAAAAAAAR0/0ZQJtMJ9B3s/s1600-h/image9.png)<!--kg-card-end: html-->

Hit the Publish button in the ribbon to publish to TFS. The Integration engine will kick in and sync the high level items to Project Server. You can check this by waiting a short while (a minute or two) and then checking the history field of the work items that you’ve sync’d.

## TF285010: Not a valid Project Server Resource

Often when you’re doing synchronization, you’ll come across this error message:

<!--kg-card-begin: html--><font face="Courier New">TF285010: The following user is not a valid Project Server resource: Colin Dembovsky. Add the Team Foundation user to the enterprise resource pool.</font><!--kg-card-end: html-->

This is actually a generic error message (which could mean that the user is not an enterprise resource or has not been added to the resource pool for the Enterprise Project) or it could be a permissions issue. Check that the TFSService account (the integration identity) has the correct permissions.

**You must also make sure that you publish the Enterprise Project**. Synchronization only works with published projects.

(Note: To get the sync engine to kick in again, I usually make a change to the title field of the work item).

Make sure you can see the following message in the history of the high level work items:

<!--kg-card-begin: html--><font face="Courier New">Project Server Sync: Successfully submitted the request to Project Server.</font><!--kg-card-end: html-->
## Approvals

So now the changes have been submitted to the Enterprise Project – if required, the Project Manager (or Timesheet approver) will need to approve the changes and publish the project.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-qh_Umvrkooo/ThWVn-dFRPI/AAAAAAAAASA/Y1ZVK-KUicU/image_thumb11.png?imgmax=800 "image")](http://lh5.ggpht.com/-ZUC52YTDwG8/ThWVm3QksmI/AAAAAAAAAR8/b92ZitpkHNo/s1600-h/image19.png)<!--kg-card-end: html-->

If you log onto your PWA, you’ll see some task approvals waiting (in this case it’s a New Task Request). Approve the requests (you can preview if you want to and you can add comments when approving). Once you’ve approved the tasks, you’ll get **approvals again**. This time there are “edits” to the tasks – hours worked, work item type and other fields. Again, you can preview if you want to – make sure you approve this 2nd approval!

Once you have approved the tasks (each one needs 2 approvals) you must **publish the project**! Open it in Project Professional and select File-\>Publish. Once you’ve done that, you’ll be able to see the work items in the Project Plan:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-KDRvLb4CPK4/ThWVpUBLMII/AAAAAAAAASI/ph7MklEuKxM/image_thumb13.png?imgmax=800 "image")](http://lh4.ggpht.com/-Hx7oQ9GDWnM/ThWVomY03LI/AAAAAAAAASE/z9Cc66V6A2M/s1600-h/image23.png)<!--kg-card-end: html-->

Now if you look at the history of Work Item in TFS, you’ll see a couple more edits from the sync engine. Note the approval that comes in (as well as the comment) **are from the 2nd approval** on the PWA.

## Publishing the Next Level

Now you can go back to your spreadsheet and change the Project Server Sync to yes on the next level of the hierarchy. I also added remaining work to the tasks at the same time. Once again, make sure you see the “Successfully submitted to Project Server” message in the history. Then do the double approval on the PWA and don’t forget to publish the plan!

You can also go ahead and enter predecessor information on the Project Plan to make sure that your resources are levelled and so on. Once you publish, you’ll see the changes (again the 2nd approval) on the work items in TFS. You’ll also notice the start dates coming in from the Project Plan as well as the padlock on the hierarchy – when you sync a hierarchy to Project Server, the links are locked within TFS. If you want to change the parenting, do it from the Project Plan.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-LQW1p3l0-PQ/ThWVrb8F-pI/AAAAAAAAASQ/NfTVW_XhLcM/image_thumb15.png?imgmax=800 "image")](http://lh6.ggpht.com/-5euIUUkQIK8/ThWVqq6C8ZI/AAAAAAAAASM/F_aIaTF3lck/s1600-h/image27.png)<!--kg-card-end: html-->