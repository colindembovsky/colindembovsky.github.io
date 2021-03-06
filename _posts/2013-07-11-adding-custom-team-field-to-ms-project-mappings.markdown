---
layout: post
title: Adding Custom Team Field to MS Project Mappings
date: '2013-07-11 16:26:00'
tags:
- alm
---

Note: This post is NOT about TFS and Project Server integration – this is for adding, editing and deleting work items using MS Project.

I was working with a customer who are using a custom field (called “Team”) instead of Area Path to denote “ownership” of work items to teams. You can find out how to do this on Martin Hinshelwood’s [excellent post](http://nakedalm.com/team-foundation-server-2012-teams-without-areas/). So I did the customization (the team is working off MSF Agile, and not the Scrum template, but the steps are the same). Everything was looking great.

Then their project managers (who work a lot in MS Project) started [adding work using the MS Project](http://msdn.microsoft.com/en-us/library/vstudio/dd286701.aspx). However, once they added in the User Stories, the items did not appear on the Backlogs. I realized that this is because the Backlogs are filtered to Team – any work item that does not have the Team field set (when using a custom team field) won’t show for the team.

The solution: customize the Team Project “[TFSFieldMapping](http://msdn.microsoft.com/en-us/library/ms404684(v=vs.100).aspx)” – this is the file that controls how Work Item fields map to MS Project fields.

## Customize TFSFieldMapping

First, you have to download the mapping file. Open up a VS command prompt and navigate to this folder:

    %programfiles%\Common Files\microsoft shared\Team Foundation Server\11.0

(This is a different folder than previous versions of TFS for some strange reason).

Run the following command:

    TFSFieldMapping download /collection:CollectionURL /teamproject:ProjectName /mappingfile:MappingFile

where collectionURL is the URL to your collection, ProjectName is the name of your Team Project and MappingFile is the file you want to create.

Now edit the file in your favourite editor and add the following line:

    <mapping workitemtrakingfieldreferencename="”Custom.Team”" projectfield="”pjTaskText27”" projectname="”Team”"></mapping>

Of course the reference name is the reference name of your custom team field. The ProjectName attribute is a friendly name for the field – this will actually be the column name in Project. The project field is an unused field in the project plan (to see these, open Project and add a new column to the plan – the fields that display here are the fields that you need to use for the mapping – see the below picture).

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-YO8b1UWFwBM/Ud5eMpZLwcI/AAAAAAAAA9Y/oTbq4HWqSz4/image_thumb.png?imgmax=800 "image")](http://lh5.ggpht.com/-HI8vfnFKN1U/Ud5eMMY-deI/AAAAAAAAA9Q/eLSwtugJHZk/s1600-h/image%25255B2%25255D.png)<!--kg-card-end: html-->

Now you can upload the mapping file again:

    TFSFieldMapping upload /collection:CollectionURL /teamproject:ProjectName /mappingfile:MappingFile

Now when you open up a project plan, you’ll be able to add the “Team” field:

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-OUcWBx6mOAc/Ud5eNqdzJpI/AAAAAAAAA9o/u0_OHkw4Uh8/image_thumb%25255B2%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-CJ4lX0LAfSs/Ud5eNKngoXI/AAAAAAAAA9g/AwkmimP_8eM/s1600-h/image%25255B6%25255D.png)<!--kg-card-end: html-->

The column will even get its values from your global list:

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-Y_ROdhDDhKY/Ud5eOk5PYZI/AAAAAAAAA94/O6acWtbUUsc/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-qFD3Fx38rQg/Ud5eOEar87I/AAAAAAAAA9w/H1tTlKehc3E/s1600-h/image%25255B9%25255D.png)<!--kg-card-end: html-->

Now when you add work using MS Project, you can select the Team and the work will appear on the Backlogs accordingly. Of course this technique will work for any other fields too.

Happy Mapping!

