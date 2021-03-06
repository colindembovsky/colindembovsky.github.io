---
layout: post
title: Getting Results from Backlog Overview Report in TFS 2013 Preview
date: '2013-08-28 17:19:00'
tags:
- alm
---

One of my favourite reports in TFS is the Backlog Overview (Scrum) or User Story Overview (Agile). So after installing and playing with TFS 2013 Preview, I went to see what the report looks like.

What I found wasn’t pretty: though I could verify that there was data in the warehouse, the report stubbornly refused to show any data.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-JxBhrUErPx8/Uh2yk24YcII/AAAAAAAABCA/PTwdcR-hfgM/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-d0_Lac10QwE/Uh2ykKy9RVI/AAAAAAAABB4/OLKW0qAGAB0/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

I thought that something was broken with my warehouse, so I dug into the rdl file and into the warehouse database. I could see data, but one of the queries wasn’t returning any data. It turns out the query is looking for the “root” level of “deliverables”. This defaults to the set (PBI, User Story, Requirement) which would work perfectly in the out-of-the-box 2012 templates. But the 2013 templates now root “higher up” in Features. So you have to add Feature to the list. Here’s how to do it:

1. Browse to the reports folder root (usually this is [http://server/reports](http://server/reports)) where server is the name of your TFS box. Now navigate through TfsReports to the collection and team project folder where your “Backlog Overview” report is:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-kM5yH2KlNKY/Uh2ymekwNZI/AAAAAAAABCQ/0vBuP4XwRTQ/image_thumb%25255B5%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-J-SvW_Qv_qk/Uh2ylvXlbSI/AAAAAAAABCI/0Por7daq1nI/s1600-h/image%25255B11%25255D.png)<!--kg-card-end: html-->

2. Hover your mouse over the “Backlog Overview” report and click the arrow to expand the menu. Select “Manage”.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-BYcRDlkO2J8/Uh2ynh6szEI/AAAAAAAABCg/JT-2VVE7nVg/image_thumb%25255B6%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-h_RuPpvEW4g/Uh2ymwcKigI/AAAAAAAABCY/u0-EYlPLFMc/s1600-h/image%25255B14%25255D.png)<!--kg-card-end: html-->

3. Click on the Parameters tab on the left and find the parameter called “DeliverableCategory”. Add “Feature” to the list. **Don’t forget to scroll down and press the “Apply” button!**

<!--kg-card-begin: html--> [![image](http://lh3.ggpht.com/-L3SYRnD1Jaw/Uh2ypmJP56I/AAAAAAAABCw/wP1h6EsXPj0/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-Ub9fsKHsWTI/Uh2yoq_lvmI/AAAAAAAABCo/WxcCDYwpACA/s1600-h/image%25255B7%25255D.png)<!--kg-card-end: html-->

Voila! You now have data when you browse to the report. The PBIs are grouped under their respective Features, which is a nice bonus.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-2SNZg-v38rE/Uh2yrSt17LI/AAAAAAAABC8/XpGu_axd7Lw/image_thumb%25255B8%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-2zhM0DW9DuM/Uh2yqTSx9yI/AAAAAAAABC4/qy_Mob-nc9k/s1600-h/image%25255B18%25255D.png)<!--kg-card-end: html-->

Happy reporting!

