---
layout: post
title: 'New Feature: Lightweight Query Charts in Web Access'
date: '2013-09-09 22:40:00'
tags:
- alm
---

I have installed [TFS 2013 RC](http://www.microsoft.com/visualstudio/eng/2013-downloads). I upgraded my TFS Express (that I use for mucking around with code) from TFS 2012.3 and everything went smoothly. I then opened up Web Access and was pleased to see one of the best features yet for TFS work items: lightweight charts.

These charts allow you to quickly and easily create visualizations against your work item queries. Here is a “dashboard” against a query that lists “Tasks” in my Team Project:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-5hDzIXCKGQc/Ui3PtC_Dq2I/AAAAAAAABD8/MMNMTbeGSFs/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-nZcBnGFzlGc/Ui3PsaeRsXI/AAAAAAAABD0/s9NkwPtEFXw/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->
## Creating Charts

Creating charts is really simple. Navigate to Web Access and click “WORK” and then “Queries” to go to the query hub. Select or create create a query. Make sure you add any columns that you want to group by onto the query – for example, State, Assigned To or Iteration Path. Once you’ve saved your query, click on the “Charts” tab:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-v9HqiR3C8fc/Ui3Pumx6AmI/AAAAAAAABEM/X6dT9wGYYms/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-lsjTEJlqih4/Ui3Ptz66k8I/AAAAAAAABEE/WTkLMUHWPhs/s1600-h/image%25255B7%25255D.png)<!--kg-card-end: html-->

Now enter a name, select the chart type and the field to group by. For example, here I am doing a pie graph grouped by “State”:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-kYnMTM9IAlk/Ui3PwC6Ei8I/AAAAAAAABEc/ElSIby_MvEM/image_thumb%25255B5%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-OwrnxvXhJbI/Ui3Pve2V52I/AAAAAAAABEU/vCz9HSPzwY0/s1600-h/image%25255B11%25255D.png)<!--kg-card-end: html-->

You can also make a Pivot table – this allows you to select rows and columns. For example, here’s one showing Assigned To vs State:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-s4ESMhJZND0/Ui3PxelnZeI/AAAAAAAABEs/zlUqNxKDTDk/image_thumb%25255B7%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-vA_9dr25AUw/Ui3PwquOZnI/AAAAAAAABEk/__b8LRwHWE4/s1600-h/image%25255B15%25255D.png)<!--kg-card-end: html-->
## Limitations

Unfortunately, you can only use “count of work items” as the value setting. I tried to see if I could add “Remaining Work” and show remaining work per user, but no dice at the moment. We’ll have to wait for a future update to get this sort of functionality.

## Wish List

I’d love to see the ability to “Favourite” one of the charts so that it appears on the landing page. It would also be nice if you could edit the colours, add filters and email the charts (or at least a chart link). Also, you can’t drag-and-drop to re-order the charts. We’ll have to see what the product team gets time to actually squeeze into this feature.

In the meantime, happy charting!

