---
layout: post
title: Running Lab Management with XP SP3 Machines
date: '2011-06-16 20:24:00'
tags:
- labmanagement
---

If you’re running Lab Management with lab machines with XP SP3, you may run into a problem where the Test Agent is never ready for running tests.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-zzOipzaI7HI/TfnocA_gTbI/AAAAAAAAAQo/no1SmH4izqE/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-5JMaUoD2vsQ/TfnobWAUwyI/AAAAAAAAAQk/oxtyWf30Ucs/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

The environment was starting up fine and configuring both network isolation as well as the workflow agent. However, the Test Capability was continually not ready – the event logs had a message like this:

Unable to connect to the controller on 'TFSMachine.Domain:6901'. The agent can connect to the controller but the controller cannot connect to the agent because of following reason: An error occurred while processing the request on the server: System.IO.IOException: The write operation failed, see inner exception. ---\> System.ComponentModel.Win32Exception: **The message or signature supplied for verification has been altered.**

Scratching around led me to this rather helpful blog site: [Troubleshooting Guide for Visual Studio Test Controller and Agent](http://social.msdn.microsoft.com/Forums/en-US/vststest/thread/df043823-ffcf-46a4-9e47-1c4b8854ca13/). The problem is mentioned there – it’s a problem with a Windows hotfix (KB968389) which you simply need to uninstall from the XP Lab machine. Reboot and you’re good to go.

Happy Labbing!

