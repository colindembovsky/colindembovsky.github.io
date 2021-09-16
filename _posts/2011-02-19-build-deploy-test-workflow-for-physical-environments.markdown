---
layout: post
title: Build-Deploy-Test Workflow for Physical Environments
date: '2011-02-19 21:12:00'
tags:
- build
---

This week, Darshan Desai published a [post](http://blogs.msdn.com/b/lab_management/archive/2011/02/16/running-build-deploy-test-workflow-on-physical-environments.aspx) about a build-deploy-test workflow for Physical environments. The solution is entirely XAML based – you don’t need any custom assemblies. However, the design-time experience is not as rich as the wizard that you get when you do a Lab workflow for build-deploy-test using the LabDefault.xaml template.

I’ve been working on building a workflow that includes a wizard similar to that of the Lab workflow. Seeing Darshan’s solution allowed me to iron out a few kinks in my solution, as well as overcome some of his solution’s limitations. Obviously, since this is a scenario for a Physical environment, there is no ability to do snapshots or restores - you'll have to make sure you have some clean up scripts to run pre-deployment to get to a "clean-ish" state.

## Notions Physical Build-Deploy-Test Solution

Setting up the environment is the same for my solution as it is for Darshan – you need to install and configure both a Build agent (or workflow agent, as it’s called in the Lab scenario) as well as a Test agent. Both agents need to be connected to controllers in your TFS environment. The name of the Build agent is important, since this is the way that you configure where deployment scripts are run.

If you’re going to test, you need to create a test plan with a test suite that contains tests that have automation associate with them. Also, you’ll need a “regular” build that can compile (and optionally unit test) the code as well as the dll’s that contain the automated tests (call this the “Source build”). Then you’ll need to create some automated test settings for your test plan. This test setup is exactly the same setup you’d need if you’re using the Lab workflow or Darshan’s workflow.

To use our workflow, you need the PhysicalDefaultWorkflow in source control somewhere, as well as a custom assembly. You’re controller needs to be configured to point to the folder in source control that contains this custom assembly. In contrast, Darshan’s workflow doesn’t require a custom assembly.  
Here’s a walkthrough of what my workflow looks like once you’ve configured the physical environment, the Source build and the test plan.

1. Create a new Build Definition and set the workspace, drop location, trigger and retention policy just as you would for any other build.

2. Change the Build Process Template on the Process tab to the PhysicalDefaultTemplate.xaml.

3. You’ll see the familiar “Click here to edit details…” for the Workflow Process Details argument. Clicking the button with the ellipsis will launch the Physical Workflow Parameters wizard. You’ll see a welcome screen – click next to start configuring the build.

4. On the “Select Environment” screen, select the Physical environment that you want to deploy to

<!--kg-card-begin: html-->[![clip_image002](http://lh5.ggpht.com/_d41Ixos7YsM/TV-jKpq_ovI/AAAAAAAAAO4/jPWG461VXno/clip_image002_thumb1.jpg?imgmax=800 "clip\_image002")](http://lh5.ggpht.com/_d41Ixos7YsM/TV-i61iKBAI/AAAAAAAAAO0/1qpy3bbu7dM/s1600-h/clip_image0024.jpg)<!--kg-card-end: html-->

5. On the “Configure Build” screen, configure the Source build. Here I overcome some of Darshan’s build’s limitations – you can choose a custom drop location, queue a new build or select the latest available build for your build definition.

<!--kg-card-begin: html-->[![clip_image004](http://lh5.ggpht.com/_d41Ixos7YsM/TV-jbahYV7I/AAAAAAAAAPA/xHHPJHDut5c/clip_image004_thumb1.jpg?imgmax=800 "clip\_image004")](http://lh4.ggpht.com/_d41Ixos7YsM/TV-jLX3HqiI/AAAAAAAAAO8/CbkTW5gTAjs/s1600-h/clip_image0044.jpg)<!--kg-card-end: html-->

6. The next screen is “Deployment Scripts”. Here you can configure scripts for the deployment. This screen is slightly different from the same screen in the Lab Workflow. Instead of “Role” for the deployment script, you need to target the Build (Workflow) agent to run the script on (you may see agents that are not part of the environment in this list, so make sure you select the correct agents). You can use $(BuildLocation) as a parameter for the build drop folder of the Source build. I’ve also created a Machine\_ parameter that’s similar to the Computer\_ (and InternalComputer\_) parameters of the Lab workflow. You use $(Machine\_AgentName) as a variable for the physical machine name that the agent with name AgentName resides on. For example, if you have an agent called “MyAgent” on a machine called “MyMachine”, then you can use $(Machine\_MyAgent) as the variable and when the build runs, this variable will be expanded to “MyMachine”.

7. You’ll need to configure an account to perform the deployment under. In the Lab Workflow, this is the account that the Lab Agent is configured with. This can be any account as long as it has permissions to execute the script and to the Source build drop folder. Warning: the password is not stored securely!

<!--kg-card-begin: html--> [![clip_image006](http://lh3.ggpht.com/_d41Ixos7YsM/TV-kZbR-nXI/AAAAAAAAAPI/r7dv1pGfTcU/clip_image006_thumb2.jpg?imgmax=800 "clip\_image006")](http://lh5.ggpht.com/_d41Ixos7YsM/TV-jcDplFCI/AAAAAAAAAPE/zYzRUWjp9K8/s1600-h/clip_image0065.jpg)<!--kg-card-end: html-->

8. Finally, configure testing on the “Configure Testing” screen. This screen is exactly the same as the “Configure Testing” screen in the Lab Workflow.

<!--kg-card-begin: html-->[![clip_image008](http://lh5.ggpht.com/_d41Ixos7YsM/TV-lXJ71PdI/AAAAAAAAAPQ/rbg-w9udgJ8/clip_image008_thumb1.jpg?imgmax=800 "clip\_image008")](http://lh6.ggpht.com/_d41Ixos7YsM/TV-kaL4-vsI/AAAAAAAAAPM/x9OruRIC55Q/s1600-h/clip_image0084.jpg)<!--kg-card-end: html-->

Now you can run your build! Here’s the output of one of my builds:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/_d41Ixos7YsM/TV-loSQIP4I/AAAAAAAAAPY/BTXWG9Tlxbc/image_thumb1.png?imgmax=800 "image")](http://lh6.ggpht.com/_d41Ixos7YsM/TV-lYRQRLPI/AAAAAAAAAPU/bApVU7dtPvI/s1600-h/image3.png)<!--kg-card-end: html-->