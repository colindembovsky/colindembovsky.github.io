---
layout: post
title: TFS 2013 Default Build – The GetEnvironmentVariable<T> Activity
date: '2013-10-30 20:34:00'
tags:
- build
---

If you’ve upgraded to TFS 2013, then you’ll notice that there’s a new Default Build template. In fact, to support Git repositories, the product team moved the default template into a super-secret-database-backed-folder-you-can’t-get-hold-of-place in TFS. This means that you won’t see it in the BuildProcessTemplates folder.

But the product team did make the default template quite flexible by building in pre- and post-build and pre- and post-test script arguments. To see how to use a pre-build script, refer to my post about [versioning assemblies to match build number](http://www.colinsalmcorner.com/2013/07/matching-binary-version-to-build-number.html).

So if you’re going to customize the default template, you’ll have to download it first. Once you download it, you’ll see that it’s quite a bit “smaller” than the old default template. The team has “bundled” a bunch of very fine-grained activities into higher level activities. However, this means that some of the items that existed in the 2012 default template no longer exist. For example, the AssociateChangeSetsAndWorkItems activity in 2012 returns the associated work items – but the 2013 AssociateChanges activity has no return parameter. So how do you get the associated changesets? Another example is the SourcesDirectory – this used to be available from an assign activity (which created the variable and put it into a variable) in 2012 – but there’s no variable for this value in the 2013 workflow.

How then do you get these values? I’ll show you how you can get access to them via a new Activity called “GetEnvironmentVariable”. We’ll do this for SourcesDirectory and associated changes.

## Downloading the TfvcTemplate.12.xaml Template

If you’re going to customize the workflow for the default activity, you’ll need to download it. In VS 2013, Go to the Team Explorer and click on Builds. Click “New Build Definition”. Click on the Process tab. Now expand the “Show Details” button on the right to show the template details. You’ll see a “Download” link. Click it and save the template somewhere (possibly to the BuildProcessTemplates folder?)

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-00NCvm_o7q0/UnDu1DcgQ3I/AAAAAAAABGI/C1Mr6qKBzUg/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-5OVWfqZPPkM/UnDu0K_LnqI/AAAAAAAABGA/oXyjcYarb5k/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

Now you can open the template to edit it (don’t forget to add it into Source Control!).

Once it’s open, add in your variables – I’m just scoping mine to the whole workflow, so I’ll add them with the root “Overall build process” activity. Click on “Variables” and add a string variable called “sourcesDir” and an IList variable called “associatedChangesets”.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-k1fmm_xtxDI/UnDu2GCgn8I/AAAAAAAABGY/8zFRa_42DKs/image_thumb%25255B7%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-tt6xVmnj5vM/UnDu1nGoizI/AAAAAAAABGM/StHphcJ337A/s1600-h/image%25255B15%25255D.png)<!--kg-card-end: html-->

Now find the place where you want to get the variables – in my case, I’ll go right down to the bottom of the Try-Catch in the RunOnAgent activity just after the “Publish Symbols…” activity.

**<u>BEWARE</u>** : The finally of this Try-Catch invokes a “ResetEnvironment” activity which will clear all the environment variables. If you need variables after this point of the workflow, be sure to remove this activity.

From the toolbox, under the “Team Foundation Build Activities” section, drag on a GetEnvironmentVariable activity. I made the type “String” for the 1st activity, and renamed it to “Get Sources Dir”. Then press F4 to get the properties of the activity – set the result to “sourcesDir”.

The name parameter you can get from an enumeration - Microsoft.TeamFoundation.Build.Activities.Extensions.WellKnownEnvironmentVariables. This enum has a list of all the variables you can query using the activity.

I set the value to “Microsoft.TeamFoundation.Build.Activities.Extensions.WellKnownEnvironmentVariables.SourcesDirectory”

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/--Ol0hxvXLLU/UnDu3eK08QI/AAAAAAAABGo/0k4wbjVV828/image_thumb%25255B5%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-qFLuPawXlUY/UnDu2xY8NEI/AAAAAAAABGg/709A8LdbqWY/s1600-h/image%25255B11%25255D.png)<!--kg-card-end: html-->

Now drag on another GetEnvironmentVariable activity and set the type to IList (optionally change the name). Set the result to “associatedChangesets” and the name to “Microsoft.TeamFoundation.Build.Activities.Extensions.WellKnownEnvironmentVariables.AssociatedChangesets”. (You’ll see AssociatedCommits too if you’re doing a Git build customization).

That’s all there is to it – you can now use the variables however you need to.

Happy building!

