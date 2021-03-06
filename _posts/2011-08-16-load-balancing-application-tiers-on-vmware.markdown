---
layout: post
title: Load Balancing Application Tiers on VMWare
date: '2011-08-16 04:18:00'
tags:
- tfsconfig
---

Most posts about load balancing TFS Application Tiers using [NLB](http://technet.microsoft.com/en-us/library/bb742455.aspx) use either physical servers or Hyper-V virtual servers. So you would think that you can do the same using VMWare for the Application Tiers, right?

Wrong. NLB doesn’t play nicely with VMWare out-of-the-box – actually, it’s the virtual switches that are problematic. The details have to do with port-flooding and RARP packets when you configure NLB in unicast mode – the details are not that important, but the solution is. There are two solutions to the problem: configure the VMWare virtual switch and set up NLB in unicast mode, or set up NLB in multicast mode.

At a customer that I had to work at recently, we couldn’t tinker with the virtual switch because there were other servers connected to it, so we had to take the multicast route. To do this, you need to set up a static ARP route once you’ve set up the NLB.

When setting up the NLB, make a note of the MAC address that the cluster is assigned – you’ll need both the IP address and the MAC address of the cluster to set up the static ARP entry.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-8ngYatToOv8/TklxBX35RqI/AAAAAAAAASg/P6HJLM-rBAA/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-AA0f_VbeXZU/Tklw_zLpkQI/AAAAAAAAASc/tagzTWnSsaE/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

On your router, configure the static ARP entry:

<!--kg-card-begin: html--><font face="Courier New">arp [ip] [cluster multicast mac] ARPA</font><!--kg-card-end: html-->

Once that’s done, you can configure all the nodes in the NLB cluster with an application tier only install. Configure the friendly name of the TFS url to the “Full Internet Name” from the above dialog (of course you’ll need an A record that points the friendly name to the IP address of the cluster).

Once that’s done, you’re good to go!

