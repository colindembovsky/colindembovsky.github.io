---
layout: post
title: Automated Builds–Why They’re Absolutely Essential (Part 2)
date: '2013-06-18 19:27:00'
tags:
- build
---

In my [previous post](http://www.colinsalmcorner.com/2013/06/automated-buildswhy-theyre-absolutely.html) I wrote about why you should be doing automated builds in general terms. In this post I’ll show you how TFS’s automated build engine gives you _consistency_ and _quality_ in your build processes. There are other build engines, but if you’re using TFS for source control (and/or test management and/or lab management and/or work item tracking) then Team Build makes the most sense as a build engine since it ties so many other parts of the ALM spectrum together.

TFS Team Build uses Workflow Foundation as the engine underneath the build. When you create a Team Project you get a few workflow XAML files out-the-box. For this post I’ll primarily discuss features of the DefaultTemplate11.1.xaml (the default build template).

## Environment

When you configure a Build Agent, you install it on a build server. Ideally this is some Virtual Machine that is “clean” – the only things installed on the machine are the things that you need to compile (and test) your code. No rogue libraries or experimental settings – just a clean, controlled environment.

Installing the build agent is a snap – mount the TFS install media and install TFS. Then run the Build Configuration wizard and connect to a build controller (which can reside on the build machine if you want).

## Labelling Sources During a Build

The build agent checks out the latest version of source control when it starts the build. As it does so, by default it labels the code that it checks out with the build name. This means that you can get the exact point-in-time code that the build used to compile, test and package. To see the labels, open the Source Control Explorer, find the folder that your build workspace is configured to download (in the Sources section of the build workflow), right-click and select “View History”. Then click on the Labels tab.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-MxXvEL4K9R0/UcA1Y78AXRI/AAAAAAAAAzk/j4KEDZDvPR8/image_thumb1.png?imgmax=800 "image")](http://lh5.ggpht.com/-2oc8o8alSrU/UcA1XfmLm6I/AAAAAAAAAzc/0QZcvAe3qWE/s1600-h/image3.png)<!--kg-card-end: html-->

_In the above picture you can see the build reports on the left, and the labels for the root folder of the build workspace on the right._

If you don’t want the build to label the sources on each build, then go to the Process tab of your build definition, expand the Advanced parameters and set “Label Sources” to false.

## Perform Code Analysis During a Build

Most of us developers know we should be doing code analysis, but few teams that I work with actually do it. Most of the time it comes down to the fact that it’s hard to monitor. However, if you include code analysis as part of your build, you’ll easily be able to track Code Analysis over time.

If you have particular projects that you care about and don’t want to run Code Analysis on all projects, then you can configure that in the Build. The default build template sets “Perform Code Analysis” to “AsConfigured” which means if a project is configured to do code analysis on build, then it does so.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-3w9Tl5usdIE/UcA1bukCT7I/AAAAAAAAAz0/1UzlgBY0bpI/image_thumb3.png?imgmax=800 "image")](http://lh5.ggpht.com/-EagqFV4mlsg/UcA1aBcY59I/AAAAAAAAAzs/hS1sApAgDb8/s1600-h/image7.png)<!--kg-card-end: html-->

Of course you can set the Code Analysis to “Never” or “Always” too.

And as easy as that you now have Code Analysis as part of your build process:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-YxyFJbZUhBc/UcA1eZQoUzI/AAAAAAAAA0E/y7acUNEYVm0/image_thumb5.png?imgmax=800 "image")](http://lh5.ggpht.com/-7NdOwTOYSc8/UcA1czOlurI/AAAAAAAAAz8/cvTmMsm9ep4/s1600-h/image11.png)<!--kg-card-end: html-->
## Layer Validation

If you’ve got Visual Studio Ultimate, you’ll be able to draw Layer Diagrams. These diagrams allow you to visualize (and validate) layering within your architecture. Team Build can validate layering when building – all you have to do is right click your modelling project (that contains your layer diagrams) in the Solution Explorer, select Properties and set the “Validate Architecture” property to true.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-dsDEEt4ywig/UcA1g6t0lHI/AAAAAAAAA0U/eHl355SF6rk/image_thumb13.png?imgmax=800 "image")](http://lh6.ggpht.com/-2r-pW6LPNlw/UcA1ftMHdEI/AAAAAAAAA0M/2ZIbnP61Yew/s1600-h/image29.png)<!--kg-card-end: html-->

As long as this project is part of the solution(s) being built, you’ll get layer validation as part of your build.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-UyPGgc2HZ8Q/UcA1jrJWcVI/AAAAAAAAA0k/FlXo3t5rsn8/image_thumb21.png?imgmax=800 "image")](http://lh6.ggpht.com/-arwaZV04ERg/UcA1iCcRySI/AAAAAAAAA0c/tB3sAfBnpzQ/s1600-h/image41.png)<!--kg-card-end: html-->

_Oh dear – someone broke our layering!_

## Symbols

You should never deploy pdb files (symbol files) to production environments. But there are times when you’ll need the symbols files – for example remote debugging, for IntelliTrace or for Manual Test Coverage (see below). Team Build effortlessly publishes your symbols to a network share and indexes them for you, so you never have to think about them or hunt for them again.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-wXwwboWzDNI/UcA1l1ear8I/AAAAAAAAA00/LNIYw-SP6W8/image_thumb28.png?imgmax=800 "image")](http://lh5.ggpht.com/-2aCt6IFSzKo/UcA1kgPq6YI/AAAAAAAAA0s/_x5Oc8HSLjM/s1600-h/image57.png)<!--kg-card-end: html-->

_Configuring Symbols and indexing on a build – the build creates a folder structure, so just supply the root folder and the build takes care of the rest._

## Unit Testing and Code Coverage

Having a automated build without unit tests is like brushing your teeth without toothpaste. Once you’ve got a build in place, add unit tests and code coverage. This will increase the quality and consistency of your releases exponentially.

So let’s assume you have unit tests. You can easily configure Team Build to run the tests and perform code coverage. Set the automated test settings (you can have multiple) appropriately. By default the discovery filter is \*\*\*test\*.dll (which is any dll with the word “test” in it in any subdirectory). Click on “Edit” to enable Code Coverage and you’re done. I’ve even configured a build engine to run QUnit js files to test JavaScript in my web projects! Of course you can add category filters too if you want to filter which tests the build should be running.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-NBV8fE80xbI/UcA1oWGA1JI/AAAAAAAAA1E/gtZq9XM_3_M/image_thumb7.png?imgmax=800 "image")](http://lh5.ggpht.com/-s9krhHh0A-8/UcA1nHtcUXI/AAAAAAAAA08/y1XQVtAfLQw/s1600-h/image15.png)<!--kg-card-end: html-->

Besides being a metric for each individual build, the pass/fail rates and coverage percentages go into the TFS Warehouse so that you can report off them and trend them.

If you’re not using the MSTest framework and you’re using nUnit or XUnit or some other framework and you have a corresponding Test Adapter in VS for running your unit tests within VS, then make sure you install the same Test Adapter on your build machine to enable it to run those tests during the build.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-cOZSAbPjctg/UcA1qx8H23I/AAAAAAAAA1U/5XoerYOGFcY/image_thumb9.png?imgmax=800 "image")](http://lh5.ggpht.com/-1Yy9zPHhcb4/UcA1pVGDShI/AAAAAAAAA1M/giN9IebOrl8/s1600-h/image19.png)<!--kg-card-end: html-->

_This build output shows a failed test._

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-Xg7EeJeNDwk/UcA1tQ-MV8I/AAAAAAAAA1k/WW26xDUtrxw/image_thumb11.png?imgmax=800 "image")](http://lh5.ggpht.com/-iVtU6BpsH1E/UcA1r_bVRHI/AAAAAAAAA1c/jMwwoF_aaq8/s1600-h/image23.png)<!--kg-card-end: html-->

_That’s better – a 100% pass rate. Looks like the coverage is a little on the low side though…_

## Code Coverage for Manual Tests

At present this only works for Web Applications running in IIS. Get your testers to run test cases out of Microsoft Test Manager (MTM) against your test servers, and then enable the “Code Coverage” diagnostic adapter. You’ll have to tell it where to find the symbols files (which you hopefully configured on your build anyway) and you’re good to go.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-wQtsQYdrYzg/UcA1v71NKZI/AAAAAAAAA10/fyH0m5N80f4/image_thumb25.png?imgmax=800 "image")](http://lh4.ggpht.com/-3TeK4EJjZwM/UcA1uRVOaGI/AAAAAAAAA1s/fARXyePvFJU/s1600-h/image49.png)<!--kg-card-end: html--><!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-kwJih5m7A4Q/UcA1yeeEJJI/AAAAAAAAA2E/1tx7O4Ec5yA/image_thumb27.png?imgmax=800 "image")](http://lh6.ggpht.com/-3d6HeFG1gJQ/UcA1wwvaNiI/AAAAAAAAA18/HW7mPi8HoGM/s1600-h/image53.png)<!--kg-card-end: html-->

_Setting the Code Coverage diagnostic adapter (and the path to the symbols) in the Test Settings section of a Lab Environment._

The great thing about Team Build is that the manual code coverage is fed back onto the build report as testers execute their manual tests. Each time a manual test run is completed, the build report is updated.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-iuDMg57R-x4/UcA11FTASnI/AAAAAAAAA2U/DOFhA1XMWsY/image_thumb30.png?imgmax=800 "image")](http://lh6.ggpht.com/-qx4dvkwOkEM/UcA1zmG7WfI/AAAAAAAAA2M/IKkDFNdXWpA/s1600-h/image61.png)<!--kg-card-end: html-->

_This build report has been updated to show an additional test run (the manual test run) and the coverage has been merged into the total coverage (so it’s showing total unit test plus manual test coverage)._

## Build Reports – or what the heck is in this build?

Get your developers into the habit of associating checkins with work items. By default, the build lists all associated checkins between “good builds”. (The last good build is the last build that was successful – no compiler errors or test failures). If those checkins are associated with work items, the work items get associated with the builds too. That means that you can look at the build report and quickly answer the question, “What work is included in this build?”. This works for “direct” associations, such as when a developer checks in code against a Bug, but also “indirect” – when a developer checks in against a Task, the Tasks parent Product Backlog Item (or User Story) is also associated with the build.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-8aAVqdcSzaM/UcA14F-XlVI/AAAAAAAAA2k/7miAOAr8kFo/image_thumb23.png?imgmax=800 "image")](http://lh5.ggpht.com/-1ANeCujc_XM/UcA12di30eI/AAAAAAAAA2c/mZx71Eh_6ZU/s1600-h/image45.png)<!--kg-card-end: html-->

_Here we can see that Bug 82 was fixed in this build. We also see that Task 84 of PBI 83 is in this build._

Unfortunately this won’t work out-the-box for _merges_. If you queue a build that has only merges as changesets, the only changesets you’ll see will be the merges themselves. Never fear though – I created a custom build task that pulls in the merged changesets and work items into the build report. You can get it [here](http://www.colinsalmcorner.com/2013/02/custom-build-task-include-merged.html).

## Found In and Integrated In – Tracking Bugs Effectively

If you’ve got a build, and your testers specify that build number as the build they are testing, then any bug logged during testing has it’s “Found In” field automatically set. When you fix the bug and check it in, the Integrated In field is set so you know which build the bug was fixed in.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-5DbIckW9V_8/UcA16j52yyI/AAAAAAAAA20/XLg49Gi1q54/image_thumb34.png?imgmax=800 "image")](http://lh3.ggpht.com/-uCJFEdBErbg/UcA15SJFzJI/AAAAAAAAA2s/amvvVd-dKD8/s1600-h/image69.png)<!--kg-card-end: html-->

_The System Tab of the default Bug work item: the “Found In” field gets populated automatically when logging a bug from MTM (where the build under test is specified) and the “Integrated In” field gets populated automatically when you resolve the bug during checkin._

## Test Impact Analysis

Let’s say you have 100 manual tests. You run them all successfully. The developer then changes some code. Which tests should you run again? Ideally, all of them – but you may not have time to run all of them. So Team Build narrows down the list by doing _test impact analysis_. When you enable this diagnostic adapter in MTM, TFS builds a map of test vs code – it tracks which code is hit for each test the tester is executing. Then on the next build, for each **passed** test case, TFS does a lookup to see if any tests hit the code that changed since the last build. Each test that is “potentially impacted” is flagged during the build so that you can test is again to make sure the changes didn’t break the code. Of course TFS assumed you’ll re-run failed tests, so this only works against passed test cases.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-z5wz_2ixz7s/UcA19ux4vTI/AAAAAAAAA3E/-DimWuSXjK4/image_thumb32.png?imgmax=800 "image")](http://lh6.ggpht.com/-THC-HX_tcPE/UcA18PhxXbI/AAAAAAAAA28/1-dxuORn4UE/s1600-h/image65.png)<!--kg-card-end: html-->

_Two tests were impacted by changes to the code – clicking on the “code changes” link opens details about what methods changed._

## Build-Deploy-Test Workflow

I won’t go into any details on this workflow, but you get a LabDefault.xaml template out-the-box when you create a Team Project. This build doesn’t compile code – it takes the output of another build (TFS or even another engine), allows you to specify scripts that automate deployment to a Lab Environment (that you’ve set up using MTM’s Lab Manager) and even run automated tests, which could include manual test cases that you’ve converted to Coded UI Tests.

## Metrics

I’ve mentioned that the build data go into the TFS warehouse – so you can see test results, coverage and code churn over time. Then you can slice-and-dice and create dashboards and reports.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-_Muf5qZsNu4/UcA2AVGNhvI/AAAAAAAAA3U/orhSKt2KLS8/image_thumb36.png?imgmax=800 "image")](http://lh3.ggpht.com/-G6v6adV-BtA/UcA1-2ozIaI/AAAAAAAAA3M/qGxQ_aX178g/s1600-h/image73.png)<!--kg-card-end: html-->

_The out-of-the-box Build Summary report._

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-tkkIAE3ewJY/UcA2C_JxrKI/AAAAAAAAA3k/O_Imn6bUmoM/image_thumb39.png?imgmax=800 "image")](http://lh6.ggpht.com/-enJCQu5OZTY/UcA2BTuQdNI/AAAAAAAAA3c/PG3xiGujBIM/s1600-h/image81.png)<!--kg-card-end: html-->

_The out-of-the-box Build Success Over Time report._

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-yLmWSqjzG_o/UcA2FSH3FVI/AAAAAAAAA30/ZZTc4j9jYyw/image_thumb41.png?imgmax=800 "image")](http://lh6.ggpht.com/-QTtdFTJm50g/UcA2D1AiKuI/AAAAAAAAA3s/bGP72TMzk5E/s1600-h/image85.png)<!--kg-card-end: html-->

_Some of the build-specific measures available when you pivot against the TFS Cube from Excel._

## Summary

There are other build engines that you can use (such as Hudson or TeamCity or Jenkins). Where they cannot compete with TFS is in the rich integration you get into work items, source control, lab management, testing and reporting. And you get most of it for free – out-the-box. In short, if you want to take a quantum leap in consistency and quality, you need to get building! The small investment up-front will be well worth it in the long run. And you’ll be able to sleep at night…

Happy building!

