---
layout: post
title: 'More DSC Release Management Goodness: Readying a Webserver for Deployment'
date: '2014-07-10 15:22:08'
tags:
- devops
---

In my previous couple of posts ([PowerShell DSC: Configuring a Remote Node to “RebootIfNeeded”](/powershell-dsc-remotely-configuring-a-node-to-rebootnodeifneeded) and [Using PowerShell DSC in Release Management: The Hidden Manual](/using-powershell-dsc-in-release-management-the-hidden-manual)) I started to experiment with Release Management’s new PowerShell DSC capabilities. I’ve been getting some great help from Bishal Prasad, one of the developers on Release Management – without his help I’d never have gotten this far!

## Meta Mofs

To configure a node (the DSC parlance for a machine) you need to create a DSC script that configures the LocalConfigurationManager. When I first saw this, I thought this was a great feature – unfortunately, when you invoke the config script, it doesn’t produce a mof file (like “normal” DSC scripts that use resources like File and WindowsFeature) so you can’t use Start-DscConfiguration to push it to remote servers. You need to invoke Set-DscLocalConfigurationManager. The reason is that a config that targets LocalConfigurationManager produces a meta.mof instead of a mof file.

If you try to run a PowerShell script in Release Management that produces a meta.mof, you’ll see a failure like this:

<!--kg-card-begin: html--><font face="Courier New">Unable to find the mof file. Make sure you are generating the MOF in the DSC script in the current directory.</font><!--kg-card-end: html-->

Of course this is because Release Management expects a mof file, and if you’re just producing a meta.mof file, the invocation will fail.

We may see support for meta.mofs in future versions of Release Management (hopefully sooner than later) but until then the workaround is to make sure that you include the LocalConfigurationManager settings inside a config that produces a mof file. Then you include two commands at the bottom of the script: first the command to “compile” the configuration – this produces a mof file as well as a meta.mof file. Then you call Set-DscLocalConfigurationManager explicitly to push the meta.mof and let Release Management handle the mof file. Here’s an example that configures a node to reboot if needed and ensures that the Webserver role is present:

    Configuration WebServerPreReqs
    {
        Node $AllNodes.where{ $_.Role -eq "WebServer" }.NodeName
        {
            # tell the node to reboot if necessary
            LocalConfigurationManager
            {
                RebootNodeIfNeeded = $true
            }
    
            WindowsFeature WebServerRole
            {
                Name = "Web-Server"
                Ensure = "Present"
            }
        }
    }
    
    WebServerPreReqs -ConfigurationData $configData
    
    # invoke Set-DscLocalConfigurationManager directly since RM doesn't yet support this
    Set-DscLocalConfigurationManager -Path .\WebServerPreReqs -Verbose
    

You can see that there is a LocalConfigurationManager setting (line 6). Line 19 “compiles” the config – given a list of nodes in $configData that includes just a single node (say fabfiberserver) you’ll see fabfiberserver.mof and fabfiberserver.meta.mof files in the current directory after calling the script. Since RM itself takes care of pushing the mof file, we need to explicitly call Set-DscLocalConfigurationManager in order to push the meta.mof file (line 22).

Now you can use this script just like you would any other DSC script in RM.

## Setting up the Release

Utilizing this script in a Release is easy – create a component that has “Source” set to your build output folder (or a network share for deploying bins that are not built in TFS build) and set the deployment tool to “Run PowerShell on Standard Environment”. I’ve called my component “Run DSC Script”.

<!--kg-card-begin: html-->[![image](/assets/images/files/1ec52f6d-cf6f-44ff-badf-db448d8537dc.png "image")](/assets/images/files/15596a48-9aab-418d-b421-c0119d9c9564.png)<!--kg-card-end: html--><!--kg-card-begin: html--><font color="#404b55">Now on the Release Template, right-click the Components node in the toolbox and add in the script component, then drag it onto the designer inside your server block (which you’ll also need to drag on from your list of servers). Then just set the paths and username and password correctly and you’re good to go.</font><!--kg-card-end: html--><figure class="kg-card kg-image-card"><img src="/assets/images/files/9b0ec259-0987-4ea3-b372-224af65cf19a.png" class="kg-image" alt="image" loading="lazy" title="image"></figure>

I’ve saved this script as “WebServerPreReqs.ps1” in the Scripts folder of my build output folder – you can see the path there in the ScriptPath parameter. My configData script is also in the scripts folder (remember the ScriptPath and ConfigurationFilePath are relative to the source path that you configure in the component). Now you can start a release!

### Inspecting the Logs

Once the release has completed, you can open the tool logs for the “Run DSC Script” component and you’ll see two “sets” of entrties. Both sets are prefixed with [SERVERNAME], indicating which node the logs pertain to. Here we can see a snippet of the Set-DscLocalConfigurationManager invocation logs (explicitly deploying the meta.mof):

<!--kg-card-begin: html--><font face="Courier New">[FABFIBERSERVER]: LCM:  [Start  Set     ]<br>[FABFIBERSERVER]: LCM:  [Start  Resource]  [MSFT_DSCMetaConfiguration]<br>[FABFIBERSERVER]: LCM:  [Start  Set     ]  [MSFT_DSCMetaConfiguration]<br>[FABFIBERSERVER]: LCM:  [End    Set     ]  [MSFT_DSCMetaConfiguration]  in 0.0620 seconds.<br>[FABFIBERSERVER]: LCM:  [End    Resource]  [MSFT_DSCMetaConfiguration]<br>[FABFIBERSERVER]: LCM:  [End    Set     ]<br>Operation 'Invoke CimMethod' complete.<br>Set-DscLocalConfigurationManager finished in 0.207 seconds.</font><!--kg-card-end: html-->

Just after these entries, you’ll see a second set of entries – this time for the remainder of the DSC invocation that RM initiates (which deploys the mof):

<!--kg-card-begin: html--><font face="Courier New">An LCM method call arrived from computer FABFIBERSERVER with user sid S-1-5-21-3349151495-1443539541-1735948571-1106.<br>[FABFIBERSERVER]: LCM:  [Start  Resource]  [[WindowsFeature]WebServerRole]<br>[FABFIBERSERVER]: LCM:  [Start  Test    ]  [[WindowsFeature]WebServerRole]<br>[FABFIBERSERVER]:                            [[WindowsFeature]WebServerRole] The operation 'Get-WindowsFeature' started: Web-Server<br>[FABFIBERSERVER]:                            [[WindowsFeature]WebServerRole] The operation 'Get-WindowsFeature' succeeded: Web-Server<br>[FABFIBERSERVER]: LCM:  [End    Test    ]  [[WindowsFeature]WebServerRole]  in 0.8910 seconds.</font><!--kg-card-end: html-->

In the next post I’ll look at using DSC to configure the rest of my webserver features as well as create a script for installing and configuring SQL Server. Then we’ll be in a good position to configure deployment of a web application (and its database) onto an environment that we know has everything it needs to run the application.

In the meantime – happy releasing!

