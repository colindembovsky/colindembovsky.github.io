---
layout: post
title: 'Fix: Release Management “Service Unavailable 503”'
date: '2014-02-07 17:01:00'
tags:
- releasemanagement
---

At a customer we installed Release Management for their TFS 2013 TFS Server. The server component installation went really smoothly – however, it was only when we installed the Client that we realized that the Release Management service was not right – we kept getting a 503 Service Unavailable error. I opened IIS and could see that the Release Management application pool was stopped. I started the app pool, but it immediately shut down. We checked the event log and saw a few obscure error messages about NullReferenceExceptions – nothing particularly helpful.

## The Problem: SharePoint

It turns out that the problem was (surprise, surprise) SharePoint. The server we were using (for a small team) was a single server TFS – data and application tier on the same machine, as well as SSRS, SSAS and SharePoint. Unfortunately, SharePoint doesn’t play well with others – specifically with other 32-bit web applications. SharePoint’s install adds a global ISAPI module, but for some reason doesn’t add a bitness filter – so if you have a 32-bit web application (like Release Management) the 64-bit SharePoint module gets loaded anyway, and this causes the 32-bit application pool to crash.

## The Fix

1. Log on to your TFS Server and open a command prompt as administrator.
2. cd to \windows\system32\inetsrv
3. Now enter:
<!--kg-card-begin: html--><font size="2" face="Courier New">appcmd.exe set config -section:system.webServer/globalModules /[name='SPNativeRequestModule'].preCondition:integratedMode,bitness64</font><!--kg-card-end: html-->

Now restart your Release Management application pool, and you’ll be good to go.

Happy releasing!

