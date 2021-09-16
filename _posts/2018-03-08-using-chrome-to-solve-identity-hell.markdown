---
layout: post
title: Using Chrome to Solve Identity Hell
date: '2018-03-08 01:33:59'
tags:
- development
---

This week at MVP summit, I showed some of my colleagues a trick that I use to manage identity hell. I have several accounts that I use to access VSTS and the Azure Portal: my own Microsoft Account (MSA), several org accounts and customer org accounts. Sometimes I want to open a release from my 10th Magnitude VSTS account so that I can grab some tasks to put into CustomerX VSTS release. The problem is that if I open the 10M account in a browser, and then open a new browser, I have to sign out of the 10M account and sign in with the CustomerX account and then the windows break… identity hell.

At first I used to open InPrivate or Incognito windows. That gave me the ability to get to 4 different profiles: IE and IE InPrivate, Chrome and Chrome Incognito. But then my incognito windows don't have cached identities or history or anything that I like to have in my browser. Hacky - very hacky.

## Solution: Chrome People

About 2 years ago I stumbled onto Chrome People (or Profiles). This really simple "trick" has been fantastic and I almost never open Incognito anymore. In the upper right of the Chrome chrome (ahem) there is a little text that tells you what your current "person" is:

<!--kg-card-begin: html-->[![image](/assets/images/files/41ad0268-687f-4d26-ab58-115081d1a475.png "image")](/assets/images/files/c5684a4c-045c-4878-a9cf-d4ca02e208e3.png)<!--kg-card-end: html-->

Click that text to open the People hub:

<!--kg-card-begin: html-->[![image](/assets/images/files/9ff3f10d-52c4-436e-8fe6-192622845d0c.png "image")](/assets/images/files/b046296c-4e92-4ee2-af41-9c5438d179e1.png)<!--kg-card-end: html-->

Here you can see that I have 5 People: ColinMSA, 10M, AdminNWC and NWC and another customer profile. To switch profiles, I just click on the name. To add a person, just click "Manage people".

<!--kg-card-begin: html-->[![image](/assets/images/files/315ab52b-f5f4-458c-bcc7-2f2ea07769e2.png "image")](/assets/images/files/bfec3555-7347-480e-86b9-4e19d31a77dc.png)<!--kg-card-end: html-->

I can easily add a new person from this view - and I can assign an icon to the person.

When you create a new person, Chrome creates a shortcut to that person's browser on the desktop. I end up clicking on that and adding it to my taskbar:

<!--kg-card-begin: html-->[![image](/assets/images/files/735cf97a-8765-4dd3-8668-8c71c286a4dc.png "image")](/assets/images/files/7d1b015a-7023-4d2e-aeae-13987165781f.png)<!--kg-card-end: html-->

If I want to open up the Azure Portal or VSTS using my MSA, I click the ColinMSA icon and I'm there. If I need to open my customer VSTS or Portal, I just click that icon. Each window is isolated and my identities don't leak. Very neat, very clean. Under the hood, the shortcuts just add a small arg to the Chrome.exe launcher:

<!--kg-card-begin: html--><font face="Courier New">--profile-directory="Profile 1"</font><!--kg-card-end: html-->

. The first profile is Default, the second is Profile 1, the third Profile 2 and so on.

## Final Thoughts

You can also do something similar in FireFox, but I like Chrome. This simple trick helps me sort out my identity hell and I can quickly switch to different identity contexts without having to sign in and out all the time. For my MSA I sign into my Google account, but I don't do that for the other browsers. All in all it's a great way to manage multiple identities.

Happy browsing!

