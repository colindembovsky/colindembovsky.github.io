---
layout: post
title: 'Running Load Tests in Lab Management: Perfmon Issues'
date: '2011-05-16 23:59:00'
tags:
- labmanagement
---

I am preparing for the demo at DevDays, and we wanted to run a Load Test using our Lab environment. However, when trying to access the performance counters on the lab machines, I got “Access Denied” errors. That lead to some searching on why this is the case.

To access performance counters on the lab machines, you have to have 2 things in place:

1. The Remote Registry service must be running on each lab machine you want to monitor

2. The user that you’re running VS out of (or the identity of the Test Controller) needs to be an administrator on the lab machines.

## Remote Registry Service

This one’s easy – open the services snap-in and start the service (set it to start automatically if you want to).

## Permissions

This one was a little harder. In my lab, I have the computers connecting to a workgroup, so I couldn’t just add notiontraining\colind as an administrator (since the lab machines aren’t on the notiontraining domain, which is the domain I am running on). So I created a local user on each lab machine with the name “colind” and the same password as notiontraining\colind. I then added the local colind user into the local admin group.

Now I can fire up VS on my box (logged in as notiontraining\colind) and can add the performance counters to each lab machine without any issues.

Happy Load Testing!

