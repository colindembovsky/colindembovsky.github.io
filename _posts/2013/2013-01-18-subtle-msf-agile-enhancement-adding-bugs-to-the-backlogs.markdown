---
layout: post
title: 'Subtle MSF Agile Enhancement: Adding Bugs to the Backlogs'
date: '2013-01-18 20:08:00'
tags:
- alm
---

TFS’s Work Item Tracking system is amazing – the flexibility and customizability of the system is fantastic. It also allows your tool to enforce your processes – which is always a good thing for efficiency in any team!

When I install / configure / customize TFS at my customers, I usually suggest that they start with one of the three out-of-the-box Process Templates – CMMI, Scrum or Agile. Then after having run a couple of sprints or iterations using these templates, decide what changes are required and implement them slowly and over time.

So how do you decide which template to start off with? At a _very_ high level, CMMI is the most formal of the templates. Scrum and Agile are very similar in most respects, and I only recommend Scrum for Scrum purists. My template of choice for most teams is the MSF Agile Template.

## Scrum vs Agile

Scrum sort-of-noses-ahead-slightly-early-on because:

- You can see Bugs along with Product Backlog items

Agile comes-from-behind-and-smashes-Scrum-to-smithereens because:

- The dashboarding and reporting are more comprehensive out-the-box
- The bug States make more sense
- In Scrum, Bugs are Approved and Committed – what does that really mean? I prefer Active-Resolved-Closed
- Scrum Bugs can’t be “resolved” during Check-in because there is no resolved state
- Scrum Bugs can’t be verified in MTM, since there is no resolved state

So it’s really the dashboards and Bug lifecycle that pushes me to recommending Agile over Scrum.

## Bugs in Planning

One of the pain points in the Agile template is planning around bugs. I see two challenges:

1. You have a Bug that is known, and you plan to fix it in a future Iteration. To do this (including planning time and resources to do the work) you would have to create a User Story that is a “copy” of the Bug so that the User Story can be part of the backlog and you can log tasks against the bug. This does mean some duplication, and in the latest Agile template (6.1) there is a Reason when you transition from “Active” to “Resolved” called "Copied to Backlog” that is designed for this scenario.
2. You find a Bug during the Sprint and you need to fix it. Again, you’d need to “copy” the Bug to a User Story and add this into the backlog. This will surface as unplanned work, which is what you want.

Of course, you can just fix the bug without the User Story, but then you’ll have to decide where to log the tasks so that they actually appear in the backlogs.

## The Solution: Modify Agile Bugs to make them more “Scrummy”

At one of my customers, we made some minor changes to the Agile template that mitigate the “copy Bug to User Story” pain. It allows you to bring the Bugs into the backlogs (just like the Scrum template) so you end up with the best of both Agile and Scrum templates. The only down side is that if you have a lot of bugs, you can end up cluttering your backlogs, but this is a problem you’d have in the Scrum template anyway.

To do this you need to add a New state to the Bug Work Item – that will make the Bug and User Story States match more closely (New, Active, Resolved, Closed). User Stories have an extra “Removed” state, but that’s not required on the Bug. Secondly, you need to add the Bug Work Item to the Requirements Category, allowing the Product Backlog to show Bugs.

Before you start, make sure you backup as you go along (by source controlling your files) and do this on a test TFS Server before attempting this on a production server! If you get the error below, don’t panic. Check your configurations and try again. I wish there was a better description of exactly what is wrong, but unfortunately there doesn’t appear to be any more detail, so you’ll have to revert and try again.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-J-Mvk3lt_aM/UPkfD8yr8XI/AAAAAAAAAis/CWxAevsAU8A/image_thumb%25255B2%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-_s4I_h3U0GE/UPkfCs912GI/AAAAAAAAAik/nE11zNDH0mM/s1600-h/image%25255B6%25255D.png)<!--kg-card-end: html-->
## Add Story Points and the “New” State to the Bug Work Item

To edit the Bug Work Item, open it using the Process Template editor or from the command line. I’ll show you how to do this from the command line and in the XML itself. I’m going to connect to localhost (my TFS server) to the DefaultCollection and work with a TeamProject called “Code”. Of course you’ll have to change the server, collection and team project name for your scenario.

From a command line, export the Bug Work Item Template:

    witadmin exportwitd /collection:http://localhost:8080/tfs/defaultcollection /p:Code /f:Bug.xml /n:Bug<br><br>

In the &nbsp;section, copy the Story Points field from the User Story work item type or just paste in the following:

    <field name="Story Points" refname="Microsoft.VSTS.Scheduling.StoryPoints" type="Double" reportable="measure" formula="sum"><br> <helptext>The size of work estimated for implementing this user story</helptext><br></field><br>

Then look for the “Planning” group in the &nbsp;and change it to the following (to add the Story Points control):

    <column percentwidth="33"><br> <group label="Planning"><br> <column percentwidth="100"><br> <control fieldname="Microsoft.VSTS.Scheduling.StoryPoints" type="FieldControl" label="Story Points" labelposition="Left"><br> <control fieldname="Microsoft.VSTS.Common.StackRank" type="FieldControl" label="Stack Rank" labelposition="Left" numberformat="DecimalNumbers" maxlength="10" emptytext="<None>"><br> <control fieldname="Microsoft.VSTS.Common.Priority" type="FieldControl" label="Priority" labelposition="Left"><br> <control fieldname="Microsoft.VSTS.Common.Severity" type="FieldControl" label="Severity" labelposition="Left"><br> </control></control></control></control></column><br> </group><br></column><br>

Now edit the &nbsp;and &nbsp;elements of the Bug. You can simply copy/paste the &nbsp;and change the value to “New” to create the New state.

    <states><br> <state value="New"><br> <fields><br> ...<br> </fields><br> </state><br> <state value="Active"><br> ...<br><states><br></states></state></states>

Then in &nbsp;remove the &nbsp;transition and add the following:

    <transition from="" to="New"><br> <reasons><br> <defaultreason value="New"><br> <reason value="Build Failure"><br> </reason></defaultreason></reasons><br></transition><br><transition from="New" to="Active"><br> <reasons><br> <defaultreason value="Activated"><br> </defaultreason></reasons><br> <fields><br> <field refname="Microsoft.VSTS.Common.ActivatedBy"><br> <allowexistingvalue><br> <copy from="currentuser"><br> <validuser><br> <required><br> </required></validuser></copy></allowexistingvalue></field><br> <field refname="Microsoft.VSTS.Common.ActivatedDate"><br> <serverdefault from="clock"><br> </serverdefault></field><br> </fields><br></transition><br><transition from="Active" to="New"><br> <reasons><br> <defaultreason value="Deactivated"><br> </defaultreason></reasons><br> <fields><br> <field refname="Microsoft.VSTS.Common.ActivatedBy"><br> <empty><br> </empty></field><br> <field refname="Microsoft.VSTS.Common.ActivatedDate"><br> <empty><br> </empty></field><br> </fields><br></transition><br>

Now you can import the Work Item Type to make the change to your Bug Work Item:

    witadmin importwitd /collection:http://localhost:8080/tfs/defaultcollection /p:Code /f:Bug.xml<br>

## Exposing Bugs on the Backlogs

To do this, you need to edit the Categories and the CommonProcessConfiguration. First let’s export those:

    witadmin exportcategories /collection:http://localhost:8080/tfs/defaultcollection /p:Code /f:Categories.xml<br><br>witadmin exportcommonprocessconfig /collection:http://localhost:8080/tfs/defaultcollection /p:Code /f:CommonProcessConfig.xml<br><br>

Open Categories.xml and add Bugs to the Requirements Category:

    <category refname="Microsoft.RequirementCategory" name="Requirement Category"><br> <defaultworkitemtype name="User Story"><br> <workitemtype name="Bug"><br></workitemtype></defaultworkitemtype></category><br>

Open CommonProcessConfig.xml and add the “New” state to the mapping:

    <bugworkitems category="Microsoft.BugCategory"><br> <states><br> <state type="Proposed" value="New"><br> <state type="InProgress" value="Active"><br> <state type="Complete" value="Closed"><br> <state type="Resolved" value="Resolved"><br> </state></state></state></state></states><br></bugworkitems><br>

Finally, import the Categories and CommonProcessConfig files:

    witadmin importcategories /collection:http://localhost:8080/tfs/defaultcollection /p:Code /f:Categories.xml<br><br>witadmin importcommonprocessconfig /collection:http://localhost:8080/tfs/defaultcollection /p:Code /f:CommonProcessConfig.xml<br><br>

## The Results

Now you’ll be able to add Bugs on your Product Backlog, as well as do work breakdowns for Bugs in the Sprint Backlog.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-vRoqf54FmVc/UPkfGJnsqfI/AAAAAAAAAi8/A4q6v31f2Ps/image_thumb%25255B4%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-6vMxmvHL80s/UPkfEhGQinI/AAAAAAAAAiw/SEMEzDHViwU/s1600-h/image%25255B10%25255D.png)<!--kg-card-end: html-->

You’ll probably want to add Work Item Type as a Column to your Product and Iteration Backlogs (press the Column Options button shown in the figure above) or you can also edit the AgileProcessConfig file (works similarly to the CommonProcessConfig – see witadmin exportagileprocessoconfiguration for more details).

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-cHA0lNQ6C5A/UPkfH-8Km5I/AAAAAAAAAjM/sFy8RfdYha0/image_thumb%25255B6%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-N0yHRDkdwoA/UPkfG6QTB-I/AAAAAAAAAjA/OhqEAfrvG3Q/s1600-h/image%25255B14%25255D.png)<!--kg-card-end: html-->

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-DhJ19VDIoqE/UPkfJlh0NrI/AAAAAAAAAjc/4ClGvDkz_NM/image_thumb%25255B9%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-R9Q3P16rVx0/UPkfInUJoqI/AAAAAAAAAjQ/o7gPmH_xCeA/s1600-h/image%25255B19%25255D.png)<!--kg-card-end: html-->

Thanks to my good friend Theo Kleynhans for working with me on this!

Happy (product back-) logging!

**<u>UPDATE:</u>** If you want to make some subtle changes to your Task Board (including showing the Type of the “Requirement” and the ID of the Work Item) and you don’t mind an unofficial “hack” – then my good friend Tiago Pascoal has a Task Board Plugin – read about it in [this post](http://pascoal.net/2013/02/team-foundation-task-board-enhancer-version-0-6-1-released/).

