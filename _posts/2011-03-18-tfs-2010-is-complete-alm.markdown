---
layout: post
title: TFS 2010 is Complete ALM
date: '2011-03-18 21:36:00'
tags:
- alm
---

[![Slide Pic](http://lh5.ggpht.com/-lX_lsp4k8hM/Th2GfKxzsTI/AAAAAAAAASY/F4pL4aDfB0k/Slide%252520Pic_thumb%25255B1%25255D.jpg?imgmax=800 "Slide Pic")](http://lh4.ggpht.com/-gDcaqtC53vw/Th2GePbOBqI/AAAAAAAAASU/13r6pNkf0zs/s1600-h/Slide%252520Pic%25255B3%25255D.jpg)<!--kg-card-end: html-->

_(This slide shows the capabilities of TFS 2010 in 1 page. The orange blocks are TFS 2010 Features)_

Some products in the Application Lifecycle Management (ALM) space provide only slices of ALM capabilities. For example, [Subversion](http://subversion.apache.org/) (SVN) is a good source repository. [Atlassian’s Jira](http://www.atlassian.com/software/jira/) provides issue tracking (and some other modules that bolt on). Other products (like [Collabnet](http://www.collab.net/) and the [Jazz](http://jazz.net/) platform) are more comprehensive ALM offerings.

I’m not going to dig too deeply into why I think [TFS 2010](http://msdn.microsoft.com/en-us/vstudio/ff637362) outshines all of these products, but I do want to focus on two main aspects that I think put TFS ahead: comprehensiveness and integration.

## TFS 2010 is Complete (or, “TFS, You Complete Me”)

TFS isn’t _just_ Source Control. It isn’t _just_ Work Item Tracking. It isn’t _just_ Lab Management, Automated Builds, Code Analysis, Automated UI Tests, Requirements Management and Project Management, Reporting and Dashboards, Performance Profiling and Load Testing, Manual Testing and Code Coverage. It is, in fact, _all of these_ (and more).

Which leads to the second major benefit of using TFS 2010 for ALM:

## TFS 2010 is Integrated (or, “TFS, You’re One For All, And All in One”)

Unlike many other “comprehensive” ALM products, TFS was designed to provide all the ALM functionality you could want _from the ground up_. It’s not just a hodge-podge of different pieces stitched together with fragile “bridges” and “connectors” (though TFS is extensible). That’s why you can link check-ins to work items and have builds report what changesets and what work items are included in a build. That’s why you can pull a report that shows your requirements with a breakdown of tasks used to implement the requirement (and a roll-up of completed and remaining work) as well as testing effort against that requirement as well as bugs against that requirement. That’s why you can get a tester to log a bug using just a title and have the system automatically attach video of the test session, historical debug information ([IntelliTrace](http://msdn.microsoft.com/en-us/library/dd264915.aspx)), event logs and other diagnostic data in seconds – your developers will never close another bug with “no repro” again. And so on, and so on.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/_d41Ixos7YsM/TYNRxqxPFxI/AAAAAAAAAQI/Ptual4ICg5U/image_thumb%5B1%5D.png?imgmax=800 "image")](http://lh6.ggpht.com/_d41Ixos7YsM/TYNRwyJLzOI/AAAAAAAAAQE/N7yLJehFg14/s1600-h/image%5B3%5D.png)<!--kg-card-end: html-->

_(This is a screen-shot of the “Stories Overview” report – this shows your requirements (stories) together with a roll up of development effort, test progress and bug status – all in one place)_

This ability to tie work items and check-ins and builds and tests and bugs, almost effortlessly, is made possible by the fact that TFS 2010 provides all these features from a centralized repository. All source code, all work items, all test results, all reports live in a single place – meaning you can link almost anything to almost anything else – and it means you can analyse your entire ALM process from start to end (using the centralized data warehouse, of course).

This all means immense productivity gains for your development team. Add to that portals and wiki’s, ad-hoc reporting, event notifications, the Microsoft Test Manager tool for manual testing and Lab Management, integration into Visual Studio (and even other IDE’s like [Eclipse](http://www.eclipse.org/) via [Team Explorer Everywhere](http://www.microsoft.com/downloads/en/details.aspx?displaylang=en&FamilyID=af1f5168-c0f7-47c6-be7a-2a83a6c02e57)) and a host of other great process tools, and you can concentrate on creating applications quickly, continually improving on quality and enhance your process along the way.

