---
layout: post
title: 'Fix: InRelease Demo “Hangs” on Keller’s 2013 Preview VM'
date: '2013-08-26 19:02:00'
tags:
- alm
---

<u>Update 2012-09-04:</u> Brian Keller posted a fix that seems to work for this problem (so you can run the InRelease build without connecting to a physical external network).

I love [Brian Keller](http://blogs.msdn.com/b/briankel/)’s VMs. I use them as the starting point for my TFS demos (which I’ve probably done hundreds of now). Brian’s latest [VM with TFS 2013](http://blogs.msdn.com/b/briankel/archive/2013/08/02/visual-studio-2013-application-lifecycle-management-virtual-machine-and-hands-on-labs-demo-scripts.aspx) is not quite what the [2012 one](http://blogs.msdn.com/b/briankel/archive/2011/09/16/visual-studio-11-application-lifecycle-management-virtual-machine-and-hands-on-labs-demo-scripts.aspx) was (no reporting services or cube, no MS Project and a host of other things missing) but it’s enough to get started with.

One of the big [new features](http://blogs.msdn.com/b/bharry/archive/2013/06/03/visual-studio-2013.aspx) of TFS 2013 is Release Management (with the [acquisition of InRelease](http://blogs.msdn.com/b/bharry/archive/2013/07/10/inrelease-acquisition-is-complete.aspx)). An Brian provides a hands-on-lab that lets you kick the tires a bit. The lab starts with you making a small modification to the FabrikamFiber website, and then using a TFS build to trigger a deployment in InRelease. I dutifully made my change and queued the build – but the build hangs, and never completes.

<!--kg-card-begin: html-->[![clip_image001](http://lh6.ggpht.com/-2WD-HsVs950/UhsnvKWec4I/AAAAAAAABBk/1XABKuHiVGo/clip_image001_thumb%25255B1%25255D.png?imgmax=800 "clip\_image001")](http://lh4.ggpht.com/-MQluE4CaTfg/Uhsnt8OdVYI/AAAAAAAABBc/vTroxm1Vr5o/s1600-h/clip_image001%25255B4%25255D.png)<!--kg-card-end: html-->

I can run other builds successfully, and if I turn off “Trigger Deployment” in the build, the build completes. That led me to conclude that InRelease was hanging for some reason. Neither the event logs, nor the InRelease logs had any useful info.

## The Solution: Connect the VM to an External Network

I mailed Brian and he gave me a few things to try – the one that ended up working was connecting the VM to an external network. From that point on, the demo works as scripted. (Read [this post](http://blogs.msdn.com/b/virtual_pc_guy/archive/2008/01/09/using-hyper-v-with-a-wireless-network-adapter.aspx) to see how to connect your HyperV VM to your WiFi, and [this one](http://sstjean.blogspot.com/2008/09/hyper-v-give-your-virtual-machines.html) about connecting your VM to a 3G dongle).

I’m not sure why you need an external network for this – there seems to be some challenges in the networking logic of InRelease. This is just a preview release, so hopefully this gets fixed before release.

Anyway, if you’ve been having this issue while working the lab, then hopefully you can now continue to check it out.

Happy releasing!

