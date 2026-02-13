---
layout: post
title: Excel Sheet Showing Parent Items Whose Child Items Are All Closed
date: '2013-03-22 20:32:00'
tags:
- alm
---

I’ve often had the question from my customers – “I’ve got a bunch of Requirements. Some of them are Active, but all their Child Tasks are Closed. Can TFS automatically close the Parent items? Or can I at least query these Requirements out?”

You can do this with a server component that will listen to Work Item change events, query the work items and transition the states. Either you’ll download some code someone else wrote or you’ll write it yourself. As for a query, well unfortunately, you can’t do this sort of query in WIQL (to the best of my knowledge). While being really powerful, WIQL does lack things like aggregations and queries that rely on history (like show me all bugs that have been reactivated more than once). You can do these sorts of queries using the TFS API, but then you have to maintain an app and code – it gets messy.

So I’ve done some Excel “juggling” and enabled you to add some formulas to a work item query that will highlight Parents that need to be closed because all their child Tasks are closed. Unfortunately I can’t give you a simple way of implementing it because Excel doesn’t support copying formulas out of the “special” tables that the TFS Plugin creates for Work Item Queries.

Here’s the end product, showing an “Action” column that highlight Parent items whose Child Items are all closed, or Child Items that are active whose Parent items are closed:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-uu-IUV_rr7s/UUxBRLnsXTI/AAAAAAAAAqM/ZzFU12rNTu4/image_thumb1.png?imgmax=800 "image")](http://lh6.ggpht.com/-kBlKYGe6tes/UUxBP4M2IHI/AAAAAAAAAqE/N4-ZHd3b9Qg/s1600-h/image3.png)<!--kg-card-end: html-->
## Set up a Tree Query with ID, Work Item Type and State

The first thing you need to do is set up the query that returns the tree of work items. This can be any query that you want, as long as you return the ID, Work Item Type and State columns. The logic I’ll show you requires at least these 3 fields.

Now open Excel and connect to that query to pull in all your Work Items.

## The Settings Sheet

Now add a new Sheet in the workbook called “Settings”. Add the following data (you can open my [sample sheet](http://sdrv.ms/16MQpQy) and copy/paste the values in if you want):

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-GEoCShfkdZo/UUxBTdMHRaI/AAAAAAAAAqc/WsKBJ17MoUE/image_thumb3.png?imgmax=800 "image")](http://lh4.ggpht.com/-EqsAQKRXMf4/UUxBSHdfsaI/AAAAAAAAAqU/rAEWmkmX1Zk/s1600-h/image7.png)<!--kg-card-end: html-->

You’ll have to set the data in the columns appropriately:

- Parent WIT
- The Work Item types that can be parents (usually the same work item types that you have in your Requirements Category)
- Child WIT
- The child work item type (usually the same work item types that you have in your Task category)
- Parent Active States
- States of the Parent work items that you consider to be “Active” states
- Child Closed States
- States of the Child work items that you consider to be “Closed” states
- Column Count
- When you pull in a Tree Query, Excel creates a column per level in the tree. This figure could therefore change is another level in the tree appears – so just count the columns from your original query and enter the count here.
- ID Column Index
- The 1-based column index of the ID column

## Adding in the Formulas

Now go back to your query sheet. Add the following column headers to the right of your query columns (don’t worry, you’ll hide most of these anyway):

- IsParent
- ChildIndex
- ParentID
- ParentActive
- ChildActive
- ParentIDMunge
- IDMunge
- Action

Your spreadsheet should look as follows:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-EK1RTzozD8c/UUxBVi3HG7I/AAAAAAAAAqs/al0OWs_EZno/image_thumb5.png?imgmax=800 "image")](http://lh6.ggpht.com/-CfgYi0KSXnc/UUxBUmO7NGI/AAAAAAAAAqk/eCTA7gWpaOA/s1600-h/image11.png)<!--kg-card-end: html-->

Now insert the following formulas into the columns (the 1st one goes into IsParent, the 2nd into ChildIndex and so on):

=IF(COUNTIF(Setup!$A$2:$A$10,[@[Work Item Type]]), "P", IF([@[Title 1]]="","C", "P"))

=IF(H3="C", IF(H2="P", 1, I2+1), "")

=IF([@IsParent]="P", "", INDEX([#All],ROW()-[@ChildIndex]-1,Setup!$F$2))

=IF([@IsParent]="P",IF(COUNTIF(Setup!$C$2:$C$10,[@State]), "Y", "N"), INDEX([#All],ROW()-[@ChildIndex]-1,4+Setup!$E$2))

=IF([@IsParent]="C",IF(COUNTIF(Setup!$D$2:$D$10,[@State]), "N", "Y"), "")

=IF([@IsParent]="P", [@ID]&(IF([@ParentActive]="Y", "N", "Y")), [@ParentId]&[@ChildActive])

=IF([@IsParent]="P", [@ID]&[@ParentActive], [@ParentId]&[@ChildActive])

=IF(AND(COUNTIF([ParentIDMunge], [@ParentIDMunge])\>1, COUNTIF([IDMunge], [@IDMunge])=1), IF([@IsParent]="P",IF([@ParentActive]="Y", "Close Me (all children closed)", ""), ""), IF(AND([@ChildActive]="Y", [@ParentActive]="N"), "Close Me (Parent is closed)",""))

Your spreadsheet should look something like this:

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-0_1tZa_2QHE/UUxBYDhzufI/AAAAAAAAAq8/MeghQlIniRk/image_thumb7.png?imgmax=800 "image")](http://lh4.ggpht.com/-4lnO85G6O5g/UUxBWz7sBSI/AAAAAAAAAq0/mL71ifeUQ6Q/s1600-h/image15.png)<!--kg-card-end: html-->
## Clean Up

Finally, to clean up a bit, select the columns from IsParent to IDMunge and hide them, leaving only the Action column visible. You can of course filter this column to not show blanks and so on.

Perhaps there’s better ways of doing this – the primary difficulty in these formulas was the fact that you’re not only querying the row you’re in, but also the row’s “parent row” (if it has one). If you can see better ways of doing this, then please add a comment!

Here’s my [sample sheet](http://sdrv.ms/16MQpQy).

Happy querying!

