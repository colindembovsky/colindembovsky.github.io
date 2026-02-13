---
layout: post
title: Using Powershell to Replace Config Settings in Lab Management
date: '2011-06-16 20:49:00'
tags:
- labmanagement
---

So you’ve got your virtual environment running and you’re deploying your application using the Lab Default Workflow. Only, you have a config file and you need to update a connection string or something in the config file for automated tests to run properly.

Now if you’re deploying websites, for goodness sake use [WebDeploy](http://www.iis.net/download/WebDeploy) &nbsp;and the Web.Config transforms within Visual Studio to configure your web.configs as part of the WebDeploy packaging. This is by far the easiest way to deploy web applications into lab environments.

However, if you’ve got some other configs that need to be twiddled as part of the deployment, then create a PowerShell script to do your replacements.

Given an XML that looks like this:

    <span style="color: blue">&lt;</span><span style="color: #a31515">configuration</span><span style="color: blue">&gt;<br> &lt;</span><span style="color: #a31515">settings </span><span style="color: red">LogTraceInfo</span><span style="color: blue">=</span>"<span style="color: blue">False</span>" <span style="color: red">KeepConnectionAlive</span><span style="color: blue">=</span>"<span style="color: blue">True</span>"<span style="color: blue">&gt;<br> &lt;</span><span style="color: #a31515">sql </span><span style="color: red">server</span><span style="color: blue">=</span>"<span style="color: blue">dbserver</span>" <span style="color: red">database</span><span style="color: blue">=</span>"<span style="color: blue">Live</span>" <span style="color: blue">/&gt;<br> <!--</span--><span style="color: #a31515">settings</span><span style="color: blue">&gt;<br><!--</span--><span style="color: #a31515">configuration</span><span style="color: blue">&gt;</span></span></span>

Let’s say you want to replace the database property. Here’s the PS to do this:

######################################################################################  
#  
# Usage: ReplaceDatabaseInConfig.ps1 -configPath "path to config" -database "new database name"  
#  
######################################################################################  
param($configPath, $database)  
  
Write-Host "Replacing database name in config file..."  
  
# Get the content of the config file and cast it to XML  
$xml = [xml](get-content $configPath)  
$root = $xml.get\_DocumentElement();  
  
# change the db name  
$root.settings.sql.database = $database  
  
#save  
$xml.Save($configPath)  
  
Write-Host "Done!"

    <span style="color: green">###########################################################################</span>

Check this script into Source Control – make sure that you get it into your drop folder somehow (add it to a solution and mark it’s action to “Copy Always” or something along those lines).

Then it’s a simple matter of executing the script as part of your deployment. If you set the working directory to the folder where your deployment copies the script, then you can invoke it as follows:

<!--kg-card-begin: html--><font face="Courier New">cmd /c powershell .\ReplaceDatabaseInConfig.ps1 -configPath "AppCustom.config" -database "Live"</font><!--kg-card-end: html-->

In your build log, you’ll see the following:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-Tc62CFoLl04/TfntwF5AKJI/AAAAAAAAAQw/YpW_GFUSe_I/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-c0Q-urMxhoE/TfntvqukHII/AAAAAAAAAQs/mB_SyVAIydA/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

Happy testing!

