---
layout: post
title: 'Using PowerShell DSC in Release Management: The Hidden Manual'
date: '2014-07-04 21:05:08'
tags:
- devops
---

Just in case you missed it, [Release Management Update 3 CTP now supports deploying using PowerShell DSC](http://blogs.msdn.com/b/visualstudioalm/archive/2014/05/22/release-management-for-microsoft-visual-studio-2013-with-update-3-ctp1-is-live.aspx). I think this is a great feature and adds to the impressive toolset that Microsoft is putting out into the DevOps area. So I decided to take this feature for a spin!

## Bleeding Edge

\<rant\>I had a boss once who hated being on the bleeding edge – he preferred being at “latest version – 1” of any OS, SQL Server or VS version (with a few notable exceptions). Being bleeding edge can mean risk and churn, but I prefer being there all the same. Anyway, in the case of the Release Management (RM) CTP, it was a little painful – mostly because the documentation is poor. Hopefully this is something the Release Management team will improve on. I know the release is only CTP, but how can the community provide feedback if they can’t even figure out how to use the tool?\</rant\>

On top of the Release Management struggles, PowerShell DSC itself isn’t very well documented (yet) since it itself is pretty new technology. This is bleeding BLEEDING edge stuff.

Anyway, after struggling on my own for a few days I mailed the product group and got “the hidden manual” as a reply (more on this later). At least the team responds fairly quickly when MVPs contact them!

### Issues

So here’s a summary of the issues I faced:

- The DSC feature only works on domain joined machines. I normally don’t use domains on my experimental VMs, so I had to make one, but most organizations nowadays use domains anyway, so this isn’t such a big issue.
- Following the RM DSC manual, I wanted to enable CredSSP. I ran the Enable-WSManCredSSP command from the manual, but got some credential issues later on.
- The current public documentation on DSC in RM is poor – in fact, without mailing the product group I would never have gotten my Proof-of-Concept release to work at all (fortunately you now have this post to help you!)
- You have to change your DSC scripts to use in Release Management (you can’t have the exact same script run in RM and in a console – the mof compilation is invoked differently, especially with config data)

## Proof of Concept – A “Start of Release” Walkthrough

I want to eventually build up to a set of scripts that will allow me to deploy a complete application (SQL database and ASP.NET website) onto a set of “fresh” servers using only DSC. This will enable me to create some new and unconfigured servers and target them in the Release – the DSC will ensure that SQL gets installed and configured correctly, that IIS, ASP.NET, MVC and any other prerequisites get set up correctly on the IIS server and finally that the database and website are deployed correctly. All without having to install or configure anything manually. That’s the dream. The first step was to create a few DSC scripts and then to get Release Management to execute them as part of the deployment workflow.

I had to create a custom DSC resource (I may change this later) – but that’s a post for another day. Assume that I have the resource files ready for deployment to a node (a machine). Here’s the script to copy an arbitrary resource to the modules folder of a target node so that subsequent DSC scripts can utilize the custom resource:

    Configuration CopyDSCResource {
        param (
            [Parameter(Mandatory=$false)]
            [ValidateNotNullOrEmpty()]
            [String]
            $ModulePath = "$env:ProgramFiles\WindowsPowershell\Modules"
        )
    
        Node $AllNodes.NodeName
        {
            #
            # Copy the custom DSC Resource to the target server
            #
            File DeployWebDeployResource
            {
                Ensure = "Present"
                SourcePath = "$($Node.SourcePath)\$($Node.ModuleName)"
                DestinationPath = "$ModulePath\$($Node.ModuleName)"
                Recurse = $true
                Force = $true
                Type = "Directory"
            }
        }
    }
    
    CopyDSCResource -ConfigurationData $configData -Verbose
    

The last &nbsp;of the script “compiles” the DSC script into a mof file that is then used to push this configuration to the target node. I wanted to parameterize the script, so I tried to introduce the RM parameter notation, which is \_\_ pre- and post-fix (such as \_\_ModuleName\_\_). No such luck. I have to hardcode configuration data in the configuration data file.

To accomplish that I’m using configuration data for executing this script. This is standard DSC practice – however, there’s one trick. For RM, you need to put the configuration data into a variable. Here’s what an “ordinary” config data script looks like:

    @{
        AllNodes = @(
            @{
                NodeName = "*"
                SourcePath = "\\rmserver\Assets\Resources"
                ModuleName = "DSC_ColinsALMCorner.com"
             },
    
            @{
                NodeName = "fabfiberserver"
                Role = "WebServer"
             }
        );
    }

To get this to work with RM, you need to change the 1st line to this:

<!--kg-card-begin: html--><font face="Courier New">$configData = @{</font><!--kg-card-end: html-->

This puts the configuration hashtable into a variable called “$configData”. This is the variable that I’m using in the CopyDSCResource DSC script to specify configuration data (see the last line of the previous script).

Meanwhile, in RM, I’ve set up an environment (using “New Standard Environment”) and added my target server (defaulting to port 5985 for PSRemoting). I’ve configured a Release Path and now I want to configure the Component that is going to execute the script for me.

I click on “Configure Apps” –\> Components and add a new component. I give it a name and specify the package path:

<!--kg-card-begin: html-->[![image](/assets/images/files/5c33849c-66e4-4675-a050-efdc8170b396.png "image")](/assets/images/files/4054bc65-0967-415e-8af1-3752290f5216.png)<!--kg-card-end: html-->

You can access the package path in your scripts using “$applicationPath”.

Now I click on the “Deployment” tab and configure the tool – I select the “Run PowerShell on Standard Environment” tool (which introduces some parameters) and leave everything as default.

<!--kg-card-begin: html-->[![image](/assets/images/files/d3fc5510-321b-4fb7-8e06-9ce6db7c93f3.png "image")](/assets/images/files/6076c6b4-a6ef-426e-a370-e42f755475da.png)<!--kg-card-end: html-->

Now let’s configure the Release Template. Click on “Configure Apps” –\> “Release Templates” and add a new Template. Give it a name and select a Release Path. In the toolbox, right-click on the Components node and add in the DSC script component we just created. Now drag into the designer the server and into the server activity drag the DSC component. We’ll then enter the credentials and the paths to the scripts:

<!--kg-card-begin: html-->[![image](/assets/images/files/6aeee89c-c910-470a-83f4-55b9eb074f32.png "image")](/assets/images/files/613957e2-7199-4e86-a0b1-d136a71244be.png)<!--kg-card-end: html-->

Since I’m accessing a network share, I specify “UseCredSSP” to true. Both ScriptPath and ConfigurationFilePath are relative to the package path (configured in the Source tab of the component). I specify the DSC script for the ScriptPath and the config data file for the ConfigurationFilePath. Finally, I supply a username and password for executing the command. We can now run the deployment!

Create a new Release and select the newly created template. Specify the build number (either a TFS build or external folder, depending on how you configured your components) and Start it.

<!--kg-card-begin: html-->[![image](/assets/images/files/501030fa-4737-4961-8f90-7544b801754e.png "image")](/assets/images/files/0180bc56-59e7-4b3f-bf44-5e29abf12480.png)<!--kg-card-end: html-->

Hopefully you get a successful deployment!

<!--kg-card-begin: html--> [![image](/assets/images/files/5bf548b5-98d2-483c-894f-3939fb54150f.png "image")](/assets/images/files/00cc9fe9-0b3f-4778-9d78-3b7b75a511a1.png)<!--kg-card-end: html-->
## Issues You May Face

Of course, not everything will run smoothly. Here are some errors I faced and what I did to rectify them.

### Credential Delegation Failure

**Symptom:** You get the following error message in the deployment log:

<!--kg-card-begin: html--><font face="Courier New">System.Management.Automation.Remoting.PSRemotingTransportException: Connecting to remote server fabfiberserver.fab.com failed with the following error message : The WinRM client cannot process the request. A computer policy…</font><!--kg-card-end: html-->

**Fix:** In the RM DSC manual, they tell you to run an Enable-WSManCredSSP command to allow credential delegation. I have VMs that have checkpoints, so I’ve run this PoC several times, and each time I get stuck I just start again at the “clean” checkpoint. Even though this command always works in PowerShell, I found that sometimes I would get this error. The fix is to edit a group policy on the RM server machine. Type gpedit.msc to open up the console and browse to “Computer Configuration\Administrative Templates\System\Credentials Delegation”. Then click on the “Allow delegating fresh credentials with NTLM-only server authentication”. Enable this rule and then add in your target servers (click the “Show…” button). You can use wildcards if you want to delegate any machine on a domain. Interestingly, the Enable-WSManCredSSP command seems to “edit” the “Allow delegating fresh credentials” setting, not the NTLM-only one. Perhaps there’s a PowerShell command or extra argument that will edit the NTLM-only setting?

<!--kg-card-begin: html-->[![image](/assets/images/files/2c026a3a-f8fd-449b-8f99-e80eff5c98d7.png "image")](/assets/images/files/d07dc23d-e926-400a-afdb-151d3bb88f94.png)<!--kg-card-end: html-->
### Configuration Data Errors

**Symptom:** You get the following error message in the deployment log:

<!--kg-card-begin: html--><font face="Courier New">System.Management.Automation.RemoteException: Errors occurred while processing configuration 'SomeConfig'.</font><!--kg-card-end: html-->

**Fix:** I found that this message occurs for 2 main reasons: first, you forget to put your config data hashtable into a variable (make sure your line 1 is $configData = @{) or you have an error in your hashtable (like a forgotten comma or extra curly brace). If you get this error, then check your configuration data file.

### Cannot Find Mof File

**Symptom:** You get the following error message in the deployment log:

<!--kg-card-begin: html--><font face="Courier New">System.Management.Automation.RemoteException: Unable to find the mof file.</font><!--kg-card-end: html-->

**Fix:** This could mean that you’ve got an “-OutputPath” specified when you invoke your config (the last line of the config script) so that the mof file ends up in some other directory. Or you have the name of your node incorrect. I found that specifying “fabfiberserver.fab.com” caused this error in my scenario – but when I changed the name to “fabfiber” I didn’t get this error. You’ll have to try the machine name or the FQDN to see which one RM is happy with.

## Challenges

The ability to run DSC during Releases is a promising tool – but there are some challenges. Here is my list of pros and cons with this feature:

### Pros of DSC in Release Management

- You don’t have to install a deployer agent on the target nodes
- You can use existing DSC PowerShell scripts (with some small RM specific tweaks) in your deployment workflows

### Cons of DSC in Release Management

- Only works on domain machines at present
- Poor documentation makes figuring out how to structure scripts and assets to RM’s liking a challenge
- You have to change your “normal” DSC script structure to fit the way RM likes to invoke DSC
- You can’t parameterize the scripts (so that you can reuse scripts in different workflows)

## Conclusion

The ability to run DSC in Release Management workflows is great – not having to install and configure the deployer agent is a bonus and being able to treat “config as code” in a declarative manner is a fantastic feature. However, since DSC is so new (and poorly documented) there’s a steep learning curve. The good news is that if you’ve already invested in DSC, the latest Release Management allows you to leverage that investment during deployments. This is overall a very exciting feature and I look forward to seeing it grow and mature.

I’ll be posting more in this series as I get further along with my experimentation!

Happy releasing!

