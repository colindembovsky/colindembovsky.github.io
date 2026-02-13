---
layout: post
title: '2012 Lab Management Standard Environment: Configuring UI Tests Agent Identity
  Problem'
date: '2012-07-12 21:58:00'
tags:
- labmanagement
---

I am a huge fan of Lab Management. Being able to manage test rigs centrally (and, if you’re using Hyper-V, self-provisioning and self-servicing) is a huge productivity bonus.

I wanted to test out a Standard Environment (what used to be called Physical Environment in TFS 2010). I am using Brian Keller’s RC VM and another VM (WebTest) for my test machine, both of which are workgrouped (and not connected to any domain).

When configuring the environment, I came across a rather strange error. One of the wizard screens when you create a Standard Environment allows you to configure the environment for UI testing (meaning that the test agent in your rig will run in interactive mode). The dialog requests that you specify the username and password for the test agent. I assumed that this would require an account on the test machine, so I specified a local account (i.e. a WebTest account).

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-ggttrGGYagY/T_7JyOfi9BI/AAAAAAAAAa0/-jYoOIkmSac/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-pD5HLC_xO4Y/T_7JwU6rnDI/AAAAAAAAAas/33oOZUb7460/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

Unfortunately, that failed verification with the following error:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-1XfyWDeKwvc/T_7J014xXpI/AAAAAAAAAbA/1pBJyNGZIpo/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-HpXy8l2aE9k/T_7JzVqxk-I/AAAAAAAAAa8/QQ3eBOwgTyE/s1600-h/image%25255B7%25255D.png)<!--kg-card-end: html--><!--kg-card-begin: html--><font face="Courier New">Verify that the machines are accessible using the user name and password provided. The credentials provided for configuring the test agent to run as process are not valid. Provide valid credentials.</font><!--kg-card-end: html-->

That didn’t make much sense to me, since I was providing credentials that I was absolutely positive worked on the test machine.

After trying various things and searching online, I realized that you need the account for the Test Agent to be a shadow account of the one on TFS (i.e. same name and password on both machines) when you’re on a workgroup. If both machines are on a domain, you just need to make sure the domain user for the Test Agent is an admin on the test machine. Oh, and if you’re going to use the build-deploy-test workflow, make sure that identity also has read-access onto your drops folder!

Happy testing!

