---
layout: post
title: 'Hybrid Lab Workflow: Standard Lab Environment with Snapshots'
date: '2012-11-01 04:19:00'
tags:
- build
- testing
- labmanagement
---

Arguably one of the best features of TFS is Lab Management. I loved Lab Management in TFS 2010, even though it was a real pain to set up. There were a lot of moving parts and setup was tricky, but once you got SCVMM configured the rest was magic.

TFS 2012 improved Lab Management in three very significant ways:

1. If you don’t have SCVMM, you can still benefit from Lab Management by using Standard Environments without having to configure anything – just install and configure (or upgrade to) TFS and you can immediately start creating Standard Environments.
2. The Lab, Build and Test Agents were consolidated into one agent that Lab Management pushes to the test machines, so you don’t have to set up any agents manually.
3. You can now run the Build-Deploy-Test workflow against Standard Environments (without snapshots) and not just against Virtual Environments (what are now called SCVMM environments in TFS 2012).

Lab Management now becomes not only powerful, but much easier.

## Standard Environments

Standard Environments allow you to configure Labs using either physical machines or “non SCVMM” virtual machines. These could be VMWare or HyperV without SCVMM. However, one of the things I miss about the old Virtual Environment (now called SCVMM Environments) is the ability to snapshot the test machines. That way you can create a “clean” snapshot that has your test machines configured before any deployments. When the Lab Workflow kicks in, you can just restore to this clean point so that any current deployments or environment changes get reversed.

You can also create another snapshot after deployment (but before testing) so that you can re-test after any automated tests without having to redeploy or manually reset the environment to the “post-deploy-pre-test” state. For example, if you have a test that adds a Customer record, you can only run it once, since the 2nd time you run the test you’re likely to get “Duplicate” errors.

In my own day-to-day work, I use HyperV on Windows 8 and so I have to create Standard Environments (since SCVMM doesn’t support Client HyperV hosts). But I often longed for the ability to be able to restore to a clean snapshot and take a post-deployment snapshot in the Lab workflow – I am using VMs after all!

## Enter Hybrid Lab Workflow

So I started tinkering around with controlling HyperV from C# and found that you could [do it via WMI](http://msdn.microsoft.com/en-us/library/cc136992(v=vs.85).aspx). Once I was able to enumerate VMs and their snapshots, create and apply snapshots, I was ready to take the experience I had from [creating a TFS 2010 physical environment Build-Deploy-Test workflow](http://colinsalmcorner.blogspot.com/2011/02/build-deploy-test-workflow-for-physical.html) and incorporate the snapshot bits and Standard Environments. The Hybrid Lab workflow was born!

Here’s the “simplified” workflow state diagram:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-ran9DR0lnC4/UJF5Ncgm8CI/AAAAAAAAAdw/VRD1MaqAzA8/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-SgigVcjWFb4/UJF5Ldaxc1I/AAAAAAAAAdo/mRcIP4V5dNo/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

The blue blocks are what you get with the out-of-the-box workflow for Standard Environments. The orange blocks are what you get if you have an SCVMM environment, and what I added in to get the Hybrid Lab workflow.

## Disclaimer

I’ve only tested this on HyperV Client (HyperV on Windows 8). So here is my “get out of jail free” image:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-nxJGo0vtCo4/UJF5RH7lkEI/AAAAAAAAAeA/pqWjqE1z5BM/image_thumb%25255B28%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-WRgOpghk5Uc/UJF5PRMMuNI/AAAAAAAAAd4/hkzZqqBvHVg/s1600-h/image%25255B55%25255D.png)<!--kg-card-end: html-->

Also note that I may sporadically tinker around with this code a little, but don’t expect large scale support. The source code is on [hybridlabworkflow.codeplex.com](http://hybridlabworkflow.codeplex.com/), so if you really want to change things, go ahead!

## Setting up the Hybrid Lab Workflow

To set up the Hybrid Lab Workflow, download the latest binaries from [hybridlabworkflow.codeplex.com](http://hybridlabworkflow.codeplex.com/). You should have a few dll’s, PsKill.exe and PsExec.exe and the HybidLabTemplate.xaml. Here are the steps to follow for installation:

1. Copy the HybridLabTemplate.xaml to the BuildProcessTemplates folder of your Team Project (where the DefaultTemplate, UpgradeTemplate and LabDefaultTemplate.xaml files live).
2. Copy all the other files to a folder under source control.
3. Check in the files (the XAML and the dll’s and exe’s).
<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-VRkMsR5BBlY/UJF5Vz8Wj1I/AAAAAAAAAeQ/HYASsv3YT2Q/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-5_9NYYbEUs8/UJF5T9eOBgI/AAAAAAAAAeI/DxJdxwV4gv4/s1600-h/image%25255B7%25255D.png)<!--kg-card-end: html-->
1. Configure your build controller to point to the folder that contains the Hybrid Lab Workflow binaries (the folder from step 2). You will want to restart Visual Studio at this point so that the wizard can launch properly.
<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-V1crAWF2tJM/UJF5ZHOPGaI/AAAAAAAAAeg/8G1PTWimFNE/image_thumb%25255B9%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-2qVJ9X4xANk/UJF5XhBXYQI/AAAAAAAAAeY/unWJmQvSSVM/s1600-h/image%25255B20%25255D.png)<!--kg-card-end: html-->
1. **IMPORTANT** : Log into the build machine as the build service and run both PsExec.exe and PsKill.exe. You’ll see a EULA popup appear – this only happens on the machine for the login once, so you only need to do this once. If you don’t do this, the Hybrid Lab Workflow will hang. I’ll explain in my [next post](http://colinsalmcorner.blogspot.com/2012/11/developing-hybrid-lab-workflow.html) why the PsExec and PsKill are necessary at all.

## Prerequisites for creating A Hybrid Lab Workflow Build

Now you’re ready to create a build. The same “pre workflow” steps for the out-of-the-box workflow apply here, as well as a couple of Hybrid Lab Workflow specific ones:

- You need to create a standard environment for the workflow out of VMs. The VMs all need to be in HyperV and on the same host. (I’ve made the Lab Workflow extensible for other virtualization platforms like VMWare, but I’ve only implemented HyperV so far – more in another post).
- For a “clean” snapshot, snapshot the VMs AFTER the lab setup and configuration has complete (i.e. when the Environment is in the Ready state in MTM)
- You’ll need to have a build that produces bins that you want to deploy to the environment (this is called the Code build)
- You’ll need to know how to deploy (either have your commands ready or include scripts into the Code build
- If you want to run automated tests, you’ll have to have created them, linked them to test cases in a test plan and created automated test settings
- You’ll need to have an admin username and password for the VM HyperV host machine

## Creating the Build

Click on “New Build Definition” in the Builds pane of the Team Explorer in VS 2012. Name your workflow on the General tab. On the “Build Defaults” tab set the Staging Location to “This build does not copy output files to a drop location”. Then click on the Process tab.

Change the process template to HybridLabTemplate.xaml (if it’s not in the drop down, then click New, then “Select an existing XAML file” and browse to the source control location of the HybridLabTemplate.xaml).

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-JgIkbgZAdbI/UJF5dD0UBKI/AAAAAAAAAew/bzjl227XSOo/image_thumb%25255B12%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-OcwSDzyAIUA/UJF5baQnpEI/AAAAAAAAAeo/gTfRLjjKt-A/s1600-h/image%25255B25%25255D.png)<!--kg-card-end: html-->

Next you’ll want to click the ellipses that appear next to “Click here to edit details” of the Lab Process Settings parameter. This will launch the Hybrid Lab Workflow Wizard.

The Welcome page reminds you of some of the prerequisites for the workflow. Click Next.

## The Environment Page

Here you’ll select your environment from the dropdown. You’ll also see the “Restore Snapshots” checkbox. Select this if you want to restore to the “clean” state. If you do, you’ll have to specify which machines and which snapshots you want to apply.

Initially the Snapshot Information will be empty and you’ll have to connect to your virtual machine host. Click the “…” button next to the “Virtual Host Machine” textbox.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-RhadNL6jmbE/UJF5gAvkBfI/AAAAAAAAAfA/vFziFxJ8mIY/image_thumb%25255B14%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-dFyDuWeKj4U/UJF5ewvJp9I/AAAAAAAAAe4/CAc_nbkbmto/s1600-h/image%25255B29%25255D.png)<!--kg-card-end: html-->

This will open the “Connect to Virtual Host” dialog. Select a host type (at present you’ll only see HyperV). If you’re running this on the host machine, enter “localhost” for the host name and leave the rest of the boxes empty (and make sure you’re running VS as administrator). If the host is another machine, enter the host name, username, domain and password to connect to the host machine. This account must be an administrator on the host machine. Click OK.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-Z_PVpIocYAc/UJF5iqbgqyI/AAAAAAAAAfQ/jcc_RVFEJns/image_thumb%25255B17%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-UBX1_1wsKto/UJF5hcKmOOI/AAAAAAAAAfI/AbrArnoYpj8/s1600-h/image%25255B34%25255D.png)<!--kg-card-end: html-->

If all is well, you’ll see the VMs from the host that are in the Lab Environment populated in the grid. Select which snapshot you want to apply (remember this is pre-deployment) for each machine. Leave the snapshot on \<\> if you don’t want to apply a snapshot on that machine.

The final bit of information you’ll need here applies only if you plan to run coded UI tests and one of your lab machines is configured to run interactively. Enter the password for the interactive agent account (I’ll explain why you need this in a later post) – this is the same password that you used when you configured the Lab in MTM.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-rWPfoi6YjSQ/UJF5k5rZOvI/AAAAAAAAAfg/yyQ_n7Ll3R8/image_thumb%25255B19%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-7UfH6pD_Kb8/UJF5jiprjiI/AAAAAAAAAfY/1vPcR0K845w/s1600-h/image%25255B38%25255D.png)<!--kg-card-end: html-->
## The Rest of the Wizard

The “Configure Build” and “Configure Testing” pages of the wizard work exactly the same as in the Lab Default template. However, there is one more bit of info in the “Deployment Scripts” tab: the “Take a post deployment snapshot” setting. Select this if you want one, and optionally enter a prefix for the snapshot name. Note that all the VMs in the lab will get the same snapshot name when the workflow runs.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-qEynVsScyMs/UJF5nM3oKVI/AAAAAAAAAfw/w15-nDy20qE/image_thumb%25255B21%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-u5ZnUp_VMDI/UJF5l12wE_I/AAAAAAAAAfo/KHEygPUuA38/s1600-h/image%25255B42%25255D.png)<!--kg-card-end: html-->

Click Finish to close the wizard, and you’re ready to rip.

## Sample Output

Here’s a screenshot of a successful run, as well as a look at the VMs in my HyperV. You’ll see that the PostDeployment snapshot name is in the build report.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-AM-1EhL1QVA/UJF5qMfXKrI/AAAAAAAAAgA/lpoTKKY7Gas/image_thumb%25255B23%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-qW60sTNeKUE/UJF5ocfQ0BI/AAAAAAAAAf4/BX1Y0bKsWFs/s1600-h/image%25255B46%25255D.png)<!--kg-card-end: html--><!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-Ns9aBl0cC8I/UJF5tmsFK2I/AAAAAAAAAgQ/6cGfuKo_avM/image_thumb%25255B25%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-CMIRw7wOgCU/UJF5rvnpxzI/AAAAAAAAAgI/f_cioxXtU-Y/s1600-h/image%25255B50%25255D.png)<!--kg-card-end: html-->

Happy (Hybrid) Lab Workflow-ing!

