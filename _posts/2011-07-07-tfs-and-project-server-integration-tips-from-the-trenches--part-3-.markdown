---
layout: post
title: 'TFS and Project Server Integration: Tips from the Trenches (Part 3)'
date: '2011-07-07 20:15:00'
tags:
- alm
---

## Links to this series:

- [Part 1 – Considerations](http://colinsalmcorner.blogspot.com/2011/07/tfs-and-project-server-integration-tips.html)
- [Part 2 – Setup](http://colinsalmcorner.blogspot.com/2011/07/tfs-and-project-server-integration-tips_07.html)
- [Part 4 – Synchronizing Hierarchies from TFS](http://colinsalmcorner.blogspot.com/2011/07/tfs-and-project-server-integration-tips_6118.html)

## Part 3: Configuring the Bits

Now that you’ve installed the Integrator, you need to configure it. This happens at various levels:

1. Register the PWA with TFS
2. Associate a Project Collection with a registered PWA
3. Specify mappings (or use the default mapping)
4. Associate a Team Project with one (or more) Enterprise Projects
5. Mark Tasks for synchronization

You only need to configure the Project Collection \<-\> PWA once – but for each Team Project that you want to map to an Enterprise Project, you’ll need to configure a mapping (again, usually just once).

I logged into a client machine with VS 2010 SP1 installed (that’s where tfsadmin.exe comes from) as TFSSetup (my admin account for the integration).

## Registering a PWA

To register a PWA, open a VS command prompt and use the following command:

<!--kg-card-begin: html--><font face="Courier New">tfsadmin ProjectServer /RegisterPWA /pwa:<strong>pwaUrl</strong> /tfs:<strong>tfsUrl</strong></font><!--kg-card-end: html-->

where

- **pwaUrl** is the url to your PWA – like [http://server/pwa](http://server/pwa)
- **tfsUrl** is the url to your TFS services – like [http://server:8080/tfs](http://server:8080/tfs)

## TF244079 when trying RegisterPWA

If you get the following (helpful!) error:

<!--kg-card-begin: html--><font face="Courier New">TF244079: An error occurred while retrieving the URL for shared services</font><!--kg-card-end: html-->

then you probably haven’t installed the latest cumulative updates for Project Server ([Project 2010 SP1](http://blogs.msdn.com/b/project/archive/2011/05/16/project-2010-sp1.aspx) is also available – if you install SP1, make sure you run the Sharepoint Configuration Wizard after installing the SP).

## Associate a Project Collection to a Registered PWA

To map a Project Collection to a PWA, open a VS command prompt and use the following command:

<!--kg-card-begin: html--><font face="Courier New">tfsadmin ProjectServer /MapPWAToCollection /pwa:<strong>pwaUrl</strong> /collection:<strong>tpcUrl</strong></font><!--kg-card-end: html-->

where

- **pwaUrl** is the url to your PWA – like [http://server/pwa](http://server/pwa)
- **tpcUrl** is the url to the Project collection – like [http://server:8080/tfs/DefaultCollection](http://server:8080/tfs/DefaultCollection)

## Specifying Mappings

You can customize the mappings yourself if you want to deviate from the default mapping – consult the help file if you want to do this. For this example, I used the default mapping.

To specify the default mapping, use the following command:

<!--kg-card-begin: html--><font face="Courier New">tfsadmin ProjectServer /UploadFieldMapping /collection:<strong>tpcUrl </strong>/useDefaultFieldMappings</font><!--kg-card-end: html-->

where **tpcUrl** is the url to the Project collection – like [http://server:8080/tfs/DefaultCollection](http://server:8080/tfs/DefaultCollection)

## Mapping a Team Project to an Enterprise Project

Now you can map a Team Project to an Enterprise Project. Use the following command:

<!--kg-card-begin: html--><font face="Courier New">tfsadmin ProjectServer /MapPlanToTeamProject /collection:<strong>tpcUrl</strong> /enterpriseProject:<strong>epmProjectName</strong> /teamProject:<strong>teamProjectName</strong> /workItemTypes:<strong>typeList</strong> <strong>/nofixedwork</strong></font><!--kg-card-end: html-->

where

- **tpcUrl** is the url to the Project collection – like [http://server:8080/tfs/DefaultCollection](http://server:8080/tfs/DefaultCollection)
- **epmProjectName** is the name of the Project Plan on Project Server (use “” to enclose the name if it contains spaces)
- **teamProjectName** is the name of the Team Project in TFS (use “” to enclose the name if it contains spaces)
- **typeList** is the list of types you want to sync – if you specify multiple types, use “” to enclose the comma-separated list without spaces after the commas (for example, “Requirement,Task” or “User Story,Task”)
- **nofixedwork** specifies no fixed work in Project (consult the Integration help file for more info on this switch).

In the next post, we’ll look at how to synchronize work items.

