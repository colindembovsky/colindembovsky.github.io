---
layout: post
title: QFE Fixes VS 2010 SP1 Bug on Test Agents
date: '2011-06-24 15:15:00'
tags:
- labmanagement
---

Last week I was helping a customer get up and running with Lab Management. In an effort to make sure they’re up to date, I installed TFS and VS 2010 SP1 on their TFS machine as well as in all the Lab machines.

Everything was going smoothly until we started running automated tests. The tests would return as “not executed” and the error was:

<!--kg-card-begin: html--><font face="Courier New">"Attempted to access an unloaded AppDomain. (Exception from HRESULT: 0x80131014)"</font><!--kg-card-end: html-->

The strange thing was that if we turned off **all** the diagnostic adaptors, the tests ran successfully

After some investigation, we found out that this was a known bug in SP1. Yesterday, however, a [QFE was released that fixes this issue](http://blogs.msdn.com/b/lab_management/archive/2011/06/23/new-qfe-for-visual-studio-2010-testing-tools.aspx). If you’ve installed TFS and VS 2010 SP1 on your TFS and in your lab machines, make sure you apply this QFE if you want to run automated tests with diagnostic data adaptors!

Happy testing!

