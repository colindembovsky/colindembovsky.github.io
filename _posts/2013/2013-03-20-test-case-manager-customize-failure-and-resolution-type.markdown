---
layout: post
title: 'Test Case Manager: Customize Failure and Resolution Type'
date: '2013-03-20 03:54:00'
tags:
- testing
---

In Test Case Manager, you can open a test run that has failed and set the Failure and Resolution types for the failure.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-GCU-UWuNV7g/UUi0QjIpIGI/AAAAAAAAApU/sxN5-ZNBTo4/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-t4a_kS-0_WQ/UUi0PYbJc0I/AAAAAAAAApM/oRpIK3_2rUc/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

I’ve had a lot of customers ask me if it’s possible to customize the lists. Up until now, the answer has been no. However, in the middle of a bunch of improvements released in the CTPs of QU2 (Quarterly Update 2), [Brian Harry mentioned](http://blogs.msdn.com/b/bharry/archive/2013/01/30/announcing-visual-studio-2012-update-2-vs2012-2.aspx) that it is now possible to customize the lists. The CTP he was referring to is NOT A GO LIVE CTP, so rather [download CTP4](http://go.microsoft.com/fwlink/?LinkId=273878) of the update (which IS GO LIVE) if you want to try this on a production server.

The customization is only possible via the command line. And even that is hard to figure out. So here are the 4 commands that you need.

Export the Current Lists

Open a developer command prompt, and type the following commands (substituting your collection URL and team project accordingly):

<!--kg-card-begin: html--><font size="2" face="Courier New">tcm fieldmapping /export /collection:<strong>collectionURL</strong> /teamproject:<strong>Project</strong> /type:FailureType /mappingFile:FailureTypes.xml</font><!--kg-card-end: html--><!--kg-card-begin: html--><font size="2" face="Courier New">tcm fieldmapping /export /collection:<strong>collectionURL</strong> /teamproject:<strong>Project</strong> /type:ResolutionType /mappingFile:ResolutionTypes.xml</font><!--kg-card-end: html-->

This will export the two lists for you. They are pretty straight-forward and self-explanatory:

    <!--?xml version="1.0" encoding="utf-16"?--><br><testfailuretypes><br> <testfailuretype name="Regression"><br> <testfailuretype name="New Issue"><br> <testfailuretype name="Known Issue"><br> <testfailuretype name="Unknown"><br></testfailuretype></testfailuretype></testfailuretype></testfailuretype></testfailuretypes><br>

    <!--?xml version="1.0" encoding="utf-16"?--><br><testresolutionstates><br> <testresolutionstate name="Configuration issue"><br> <testresolutionstate name="Needs investigation"><br> <testresolutionstate name="Product issue"><br> <testresolutionstate name="Test issue"><br></testresolutionstate></testresolutionstate></testresolutionstate></testresolutionstate></testresolutionstates><br>

Now edit the lists, and then use the import commands:

<!--kg-card-begin: html--><font size="2" face="Courier New">tcm fieldmapping /import /collection:<strong>collectionURL</strong> /teamproject:<strong>Project</strong> /type:FailureType /mappingFile:FailureTypes.xml</font><!--kg-card-end: html-->

<!--kg-card-begin: html--><font size="2" face="Courier New">tcm fieldmapping /import /collection:<strong>collectionURL</strong> /teamproject:<strong>Project</strong> /type:ResolutionType /mappingFile:ResolutionTypes.xml</font><!--kg-card-end: html-->

That’s it – restart (not just refresh) Test Case Manager and you’re good to go.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-hy4l-RVHqcE/UUi0TiAe36I/AAAAAAAAApk/gvBaS1v1EjY/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-4Q1g78FyB44/UUi0Rg70FdI/AAAAAAAAApc/l2FlHrIvBjM/s1600-h/image%25255B7%25255D.png)<!--kg-card-end: html-->

Of course they new values appear in the Plan Results page, and though I haven’t tested it, I presume they’ll be in the warehouse too:

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-DYoCHNhi4Us/UUi0WOenaNI/AAAAAAAAAp0/14-5SeK8km4/image_thumb%25255B5%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-PQkxMGznEbQ/UUi0U-wkp9I/AAAAAAAAAps/pijAP-bwSN8/s1600-h/image%25255B11%25255D.png)<!--kg-card-end: html-->

Happy testing!

