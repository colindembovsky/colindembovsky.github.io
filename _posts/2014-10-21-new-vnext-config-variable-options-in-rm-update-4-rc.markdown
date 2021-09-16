---
layout: post
title: New vNext Config Variable Options in RM Update 4 RC
date: '2014-10-21 22:59:47'
tags:
- releasemanagement
---

[Update 4 RC for Release Management](http://www.microsoft.com/en-us/download/details.aspx?id=44555&WT.mc_id=rss_alldownloads_all) was released a few days ago. There are some [good improvements](http://support.microsoft.com/kb/2994375) – some are minor, like the introduction of “Agent-based” labels improves readability for viewing agent-based vs non-agent based templates and components. Others are quite significant – like being able to use the Manual Intervention activity and tags in vNext templates, being able to use server-drops as release source and others. By far my favorite new feature of the update is the new variable capabilities.

## Variables: System, Global, Server, Component and Action

Be aware that, unfortunately, these capabilities are **only** for vNext components (so they won’t work with regular agent-based components or workflows). It’s also unlikely that agent-based components will ever get these capabilities. I’ve mentioned before that I think PowerShell DSC is the deployment mechanism of the future, so you should be investing in it now already. If you’re currently using agent-based components, they do have variables that can be specified at design-time (in the deployment workflow surface) – just as they’ve always had.

The new vNext variable capabilities allow you to use variables inside your PowerShell scripts without having to pass them or hard-code them. For example, if you define a global variable called “MyGlobalVar” you can just use it by accessing $MyGlobalVar in your PowerShell script.

### Global Variables

Global variables are defined under “Administration-\>Settings-\>Configuration Variables”. Here you can defined variables, giving them a name, type, default value and description.

<!--kg-card-begin: html-->[![image](/assets/images/files/b03d629d-6ae1-4c0b-9481-d447a868158d.png "image")](/assets/images/files/33abda31-63d2-4440-8d2f-2faf5348e1fe.png)<!--kg-card-end: html-->

Server Variables

Server variables can be defined on vNext servers under “Configure Paths-\>Servers”. Same format as System variables.

<!--kg-card-begin: html-->[![image](/assets/images/files/c8f6bff0-ed17-4b2d-bf22-35c00de2bd18.png "image")](/assets/images/files/e6106605-d0cc-4ec8-91c1-f1ed36a93f88.png)<!--kg-card-end: html-->

Component Variables

vNext components can now have configuration variables defined on them “at design time”.

<!--kg-card-begin: html-->[![image](/assets/images/files/ff9bea76-6a78-4897-97c2-4aa42759d64b.png "image")](/assets/images/files/976326bb-0e2f-4fa9-9e79-84e2d7577fa2.png)<!--kg-card-end: html-->

You can also override values and event specify additional configuration variables when you add the “DSC” component onto the design surface:

<!--kg-card-begin: html-->[![image](/assets/images/files/02367d62-9d20-4e97-b57f-7b42e63c5ec9.png "image")](/assets/images/files/64fe1bb0-fc6e-4bba-bd68-748b02644429.png)<!--kg-card-end: html-->

Another cool new feature is the fact that ComponentName and ServerName are now dropdown lists on the “Deploy using DSC/PS” and “Deploy using Chef” activities, so you don’t have to type them manually:

<!--kg-card-begin: html-->[![image](/assets/images/files/82da646b-95e0-4066-9dc0-eee359120e3d.png "image")](/assets/images/files/4ccfae7c-03d1-4137-8253-8d5ac906eecd.png)<!--kg-card-end: html-->

All these variables are available inside the script by simple using $_variableName._ You may event get to the point where you no longer need a PSConfiguration file at all!

You can also see all your variables by opening the “Resource Variables” tab:

<!--kg-card-begin: html-->[![image](/assets/images/files/a1fc6126-479c-4194-8336-b0537a954ac7.png "image")](/assets/images/files/516ad343-1284-4999-b7ab-0581329dc5f0.png)<!--kg-card-end: html-->
### System Variables

RM now exposes a number of system variables for your scripts. These are as follows:

- Build directory
- Build number (for component in the release)
- Build definition (for component)
- TFS URL (for component)
- Team project (for component)
- Tag (for server which is running the action)
- Application path (destination path where component is copied)
- Environment (for stage)
- Stage

You can access these variable easily by simply using $_name_ (for example: $BuildDirectory or $Stage). If you mouse over the “?” icon on right of the Component or Server screens, the tooltip will tell you what variables you have access to.

<!--kg-card-begin: html-->[![image](/assets/images/files/ed79f014-2b7f-4919-a1ba-e5994537b0d2.png "image")](/assets/images/files/c11f8658-3b9d-4d10-82bb-a7817e5c646e.png)<!--kg-card-end: html-->
### Release Candidate

Finally, remember that this Release Candidate (as opposed to CTPs) is “go-live” so you can install it on your production TFS servers and updating to the RTM is supported. There may be minor glitches with the RC, but you’ll get full support from MS if you encounter any.

Happy releasing!

