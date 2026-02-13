---
layout: post
title: Using TFS ALM for Sharepoint Development (The Easy Way – with Lab Management)
date: '2012-02-10 20:46:00'
tags:
- alm
---

Visual Studio 2010 has some amazing features for Sharepoint development, like project templates, server explorers, feature and package GUIs to name a few. So you’re tasked with creating a WebPart or a Workflow – no problem, fire up VS, create a new project and you’re coding.  
However, just because you’re up and coding quickly, doesn’t mean you’re being productive (necessarily). What about requirements management? Testing? Source control? And if there’s more than 1 of you coding, what Sharepoint site do you code against? Oh wait, I forgot to mention that you need to install Sharepoint on the same machine that you have VS on to actually get the Sharepoint projects to work.

## The ChallengesThere are a few challenges that you’ll need to overcome if you want to be a good ALM citizen while doing Sharepoint dev:  

- Sharepoint and VS need to be on the same machine
- Source code needs to go somewhere other than your hard drive
- Build Automation
- Deployment
- Automated TestingFortunately, without too much work, you can address all of these challenges using TFS, and specifically Lab Management. Of course TFS covers the other ALM concerns too (like requirements management and bug tracking). I won’t focus on these too much in this article; I’ll concentrate more on the “technical challenges” bulleted above.  

## Setting up a Dev Environment using a VMInstalling Sharepoint on your local machine will allow you to at least get your applications compiling, but it’s not a sustainable solution to the SP/VS-on-the-same-machine problem. I decided to use a VM. Here are the steps I followed to set up the VM for development:  

- Install OS and join domain
- Install and configure Sharepoint
- Install VS and connect to TFS
- SnapshotNow I have a VM for Sharepoint development. I can easily duplicate this VM for other team members as they need development environments.  
Note: Because this VM is going to also become a test environment, I was careful to add my solution to Source Control. That way if I restored to a previous snapshot I don’t lose any code!  

## Build AutomationBuilding the code in TeamBuild is easy – but you’ll have to jump through some extra hoops if you want to create a WSP that can be deployed to your test Sharepoint.  
Chris O’Brien (from the Sharepoint Dev Team) wrote a series of blogs about Continuous Integration with TFS and Sharepoint (supposedly there are 5 posts, but I only found 3: [one](http://blogs.msdn.com/b/sharepointdev/archive/2011/08/25/creating-your-first-tfs-build-process-for-sharepoint-projects.aspx), [three](http://blogs.msdn.com/b/sharepointdev/archive/2011/09/22/configuring-versioning-of-assemblies-in-sharepoint-automated-build.aspx) and [five](http://blogs.msdn.com/b/sharepointdev/archive/2011/11/17/deploying-wsps-as-part-of-an-automated-build.aspx)). They’re certainly detailed – but it seemed like a lot of work to go through. I think that my Lab Management solution overcomes some of the complexities that Chris had to work through (most of which revolved around remote deployment of WSP’s).  
To get automated builds working, start off using the DefaultTemplate and get the build to compile your solution. Once you get a passed build, it’s time to create the WSP package. So open up the build template and add  
/p:IsPackage=true  
to the MSBuild arguments parameter.  

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-EG9Kiid9yR8/TzT1NEsl-cI/AAAAAAAAAWg/Bcw1WzSkl7M/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-ew0gqU5DhHs/TzT1LAaZVMI/AAAAAAAAAWY/Czz7p9kdSKY/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

However, your build is now going to fail (unless you have Sharepoint installed on your build server – which you shouldn’t!). The Sharepoint packaging requires some build targets as well as some dll’s that only get installed with Sharepoint.  
One of the most useful bits of info in Chris O’Brien’s articles is the mention of [this Powershell script](http://archive.msdn.microsoft.com/SPPwrShllTeamBuild) that will “harvest” the Sharepoint dll’s and build targets and then “install” them on your build server. I ran the script (1st on the Sharepoint dev VM), copied over the folder it created to the build server and ran the script again to install. Easy as pie. Trigger another build, and you’ll see in the drop folder a shiny new WSP package!

## DeploymentDeploying your WSP to the Sharepoint site is arguable the most difficult challenge that you’ll face. That’s why Chris O’Brien’s article gets quite complicated. Enter Lab Management.  
Since we’re working off a VM anyway, let’s bring it into Lab Management. The workflow capability (provided by the TeamBuild agent) will allow us to automate deployment not from a remote perspective, but “locally” as it were (we’re going to execute the deployment from the Sharepoint machine, not remotely from the build server). This is a massive simplification.  
These are the steps I followed to enable deployment:  

- Install the TFS Build, Lab and Test agents on the Sharepoint Dev VM
- I configured the Build and Test agents, hooking them up to existing Build and Test controllers in my TFS environment
- Compose a new Virtual Environment in Lab Management
- Bring in the Sharepoint VM and enable Deployment and Testing capabilities
<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-F0z39yVhcF0/TzT1QAwWMnI/AAAAAAAAAWw/qF2HN6XzyUc/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-WIvWKQL5Cc4/TzT1Oq2_ffI/AAAAAAAAAWo/H7ouzITP4-o/s1600-h/image%25255B9%25255D.png)<!--kg-card-end: html-->

- Use VS to deploy the package to the Sharepoint site &nbsp;by debugging the project
- In the Sharepoint Tab of the Sharepoint project properties, make sure you turn off the “Auto-retract after debug” option
<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-jSs-UwwBDqY/TzT1TIZekCI/AAAAAAAAAXA/nseLctO1QDM/image_thumb%25255B2%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-CNwzt6mgjK8/TzT1RW9RKvI/AAAAAAAAAW4/DCOxfT6DdTs/s1600-h/image%25255B6%25255D.png)<!--kg-card-end: html-->

- Create a custom page to add the WebPart in (this is only necessary for visual development, like WebParts)
- Create a Powershell script that can update the WSP (here’s [mine](https://skydrive.live.com/?cid=64A24E0938D6D062&id=64A24E0938D6D062%21312) – it’s pretty generic, so should just work)
- Add this to the Sharepoint project and set its “Copy to output directory” property to “Copy always” (this ensures that it ends up in the drop folder)

## Coded UI TestingI’m not going to go into too much detail here – I created a test plan and a test suite. I then created a test case and executed the case (with Action Recording enabled). I then added a Test Project to my Sharepoint solution and generated a coded UI test from the Test Case action recording. Then I associated the test method to the Test Case (in the Associated Automation tab of the test case). Voila – one automated test case ready to fire.  
You’ll need to open MTM and create Test Settings (automated in this case) for your Sharepoint Lab environment.  

## Lab Management WorkflowThe final step is hooking it all up in the Lab Management workflow. Create a new build and change the template to the LabDefaultTemplate. Click the ellipsis to launch the Lab Workflow Wizard.  

- Environments Tab: Select the Sharepoint environment from the list of environments
- Build Tab: Select the build that you created using the DefaultTemplate and that builds your WSP package
- Note: The coded UI test project must be part of this solution too, so that the dll’s end up in the drop folder
- Deploy Tab: check the “Deploy the build” checkbox
- Add the following 2 scripts (make sure you create the c:\deployment folder on the VM):
- cmd /c xcopy /Y $(BuildLocation)\*.\* c:\deployment
- cmd /c powershell c:\deployment\deployWSP.ps1 $(BuildLocation)
<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-JBRRCsfsYNU/TzT1WTuG6GI/AAAAAAAAAXQ/94WB3zFxwCg/image_thumb%25255B5%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-Y2Ld1FDSSMU/TzT1UX2HWOI/AAAAAAAAAXI/mLT0v8LksFg/s1600-h/image%25255B13%25255D.png)<!--kg-card-end: html-->

- Test Tab: Select the test plan, suite, configuration and settings

## ConclusionUsing Lab Management greatly simplifies the ALM aspects of Sharepoint development – automated build, deployment and testing specifically.  
Summary of Steps:  

- Create a VM and install Sharepoint and VS 2010
- Create your SP solution and _check into Source control!_
- Install and configure TFS Build, Test and Lab agents
- Compose a new Environment using the VM
- Create automated test settings for the environment
- Deploy the package by debugging the project (turn off auto-retract after debugging)
- Create test cases in a test plan
- Execute them using the Action Recording
- Turn the action recordings into Coded UI Tests
- Associate the test methods to the Test Cases
- Customize the build to produce a WSP package
- This includes “installing” Sharepoint dlls and build targets onto your build server
- Create a powershell script that can update the deployed Package from a WSP file
- Hook it all up using the Lab Management Workflow
<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-Oy3iLHxD360/TzT1ZPl6rvI/AAAAAAAAAXg/jyrem8Ncc8w/image_thumb%25255B7%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-LDp9BCuUWNc/TzT1XpF_2RI/AAAAAAAAAXY/xr4j3yGy8BE/s1600-h/image%25255B17%25255D.png)<!--kg-card-end: html-->

Happy SP dev’ing!

