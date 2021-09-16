---
layout: post
title: 'Lab Management: Configuring Workgroup Lab Machines to a TFS on a Domain'
date: '2012-08-30 04:45:00'
tags:
- labmanagement
---

I’ve installed Windows 8 on my laptop, enabled HyperV and upgraded my TFS from 2010 to 2012. Since Lab Management 2012 introduced Standard Environments, I can use my HyperV machines in lab environments right out of Windows 8. However, while I was configuring my lab, I ran into a problem. My laptop is on my work domain, and my lab machines (running on HyperV) are not joined to the domain – they’re in a workgroup.

Having the TFS on a domain and the lab machines in a workgroup can cause the following symptoms:

1. When adding the lab machine to an environment, TFS cannot push the test agent to the lab machine. The environment stalls in the “Preparing to install agent”.
2. If you install the test agent on the lab machine manually, you cannot connect to the controller on the TFS machine. The Test Agent Config Tool complains that the test controller is not accessible or the service is not running, even though you’ve verified that it is.

Here are a couple of things you can check – once I had done all of these, the environment came online without any issues.

## Shadow Accounts

Since you’re dealing with a workgroup, you’re going to need to create “shadow” accounts. These are local accounts on each machine that have the same name and password. I created a local account on my TFS machine (note: this is not a domain account – it’s a local account) and did the same on the lab machines, using the same password each time. Then make sure you add the accounts into the local administrators group.

When you configure the Test Controller on the TFS machine, use “.\username” for the logon account and the “lab service” account, where username is the name of the shadow account you created. In the figure below, you’ll see how I’ve done that using my shadow account, “labservice”.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-gmh1XpU_oFI/UD5xQXmOpNI/AAAAAAAAAcY/ffq6SCMAxWI/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-SzO2qJAk-SQ/UD5xPLbf7fI/AAAAAAAAAcQ/IGrqrMfjMIw/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->
## Network Settings on the Lab Machine

Since the Lab machine is on a workgroup, you have to make sure you can successfully communicate with the TFS machine which is on the domain. The first thing you’ll need to do is make sure that you enable File and Printer sharing on the Lab machine. If you right-click the network icon in the Taskbar, select “Open Network and Sharing Center”. Then click the link on the left, “Change advanced sharing settings”. Check that you have File and Printer sharing turned on for your “current” network profile (in the case of my lab machines, since they’re on HyperV, it’s the network connected to my virtual network).

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-DK9IN-dO-2o/UD5xURu7WvI/AAAAAAAAAco/cxajQ_5kB4M/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-yubYbEQn_Qg/UD5xTD_T3PI/AAAAAAAAAcg/NdidJf1S7f0/s1600-h/image%25255B7%25255D.png)<!--kg-card-end: html-->

You’ll also probably need to add an entry to the hosts file on the lab machine. The trick here is to add an entry using the FQDN of the TFS machine, not simply the machine name. On your TFS machine, open a command prompt and ping yourself: if your machine’s name is “mytfs” then type “ping mytfs”. This should show you the FQDN of the TFS machine – something like “mytfs.domain.local” or something. That’s the name you’ll need for the hosts entry on the lab machine. Go to the lab machine and open c:\windows\system32\drivers\etc\hosts and add an entry for the TFS machine, using the form IP &nbsp;FQDN. Here is an example from one of my lab machines:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-atbN0Klxga4/UD5xW00wd2I/AAAAAAAAAc4/pGV9rIE4l_A/image_thumb%25255B5%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-wVITFbHQ5UA/UD5xVfRNSRI/AAAAAAAAAcw/YHYHpUmD25g/s1600-h/image%25255B11%25255D.png)<!--kg-card-end: html-->

That should be it – you can now repair your environment, and you should be good to go!

Happy testing!

