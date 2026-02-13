---
layout: post
title: 'PowerShell DSC: Remotely Configuring a Node to “RebootNodeIfNeeded”'
date: '2014-06-26 16:35:19'
tags:
- development
---

I’ve started to experiment a bit with some [PowerShell DSC](http://blogs.technet.com/b/privatecloud/archive/2013/08/30/introducing-powershell-desired-state-configuration-dsc.aspx) – mostly because it’s now [supported in Release Management](http://blogs.msdn.com/b/visualstudioalm/archive/2014/05/22/release-management-for-microsoft-visual-studio-2013-with-update-3-ctp1-is-live.aspx) (in Update 3 CTP at least).

Sometimes when you apply a configuration to a node (machine), the node requires a reboot (for example adding .NET4.5 requires the node to reboot). You can configure the node to reboot immediately (instead of just telling you “a reboot is required”) by changing a setting in the node’s LocalConfigurationManager. Of course, since this is configuration, it’s tempting to try to do this in a DSC script – for example:

    Configuration SomeConfig
    {
       Node someMachine
       {
          LocalConfigurationManager
          {
             RebootNodeIfNeeded = $true
          }
       }
    }

This configuration “compiles” to a mof file and you can apply it successfully. However, it doesn’t actually do anything.

## Set-DscLocalConfigurationManager on a Remote Node

Fortunately, there is a way to change the settings on the LocalConfigurationManager remotely – you use the cmdlet Set-DscLocalConfigurationManager with a CimSession object (i.e. you invoke it remotely). I stumbled across this when looking at the documentation for [DSC Local Configuration Manager](http://technet.microsoft.com/en-us/library/dn249922.aspx) where the very last sentence says “To see the current Local Configuration Manager settings, you can use the Get-DscLocalConfigurationManager cmdlet. If you invoke this cmdlet with no parameters, by default it will get the Local Configuration Manager settings for the node on which you run it. To specify another node, use the CimSession parameter with this cmdlet.”

Here’s a script that you can modify to set “RebootNodeIfNeeded” on any node:

    Configuration ConfigureRebootOnNode
    {
        param (
            [Parameter(Mandatory=$true)]
            [ValidateNotNullOrEmpty()]
            [String]
            $NodeName
        )
    
        Node $NodeName
        {
            LocalConfigurationManager
            {
                RebootNodeIfNeeded = $true
            }
        }
    }
    
    Write-Host "Creating mofs"
    ConfigureRebootOnNode -NodeName fabfiberserver -OutputPath .\rebootMofs
    
    Write-Host "Starting CimSession"
    $pass = ConvertTo-SecureString "P2ssw0rd" -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential ("administrator", $pass)
    $cim = New-CimSession -ComputerName fabfiberserver -Credential $cred
    
    Write-Host "Writing config"
    Set-DscLocalConfigurationManager -CimSession $cim -Path .\rebootMofs -Verbose
    
    # read the config settings back to confirm
    Get-DscLocalConfigurationManager -CimSession $cim

Just replace “fabfiberserver” with your node name and .\ the script. The last line of the script reads back the LocalConfigurationManager settings on the remote node, so you should see the RebootNodeIfNeeded setting is true.

<!--kg-card-begin: html-->[![image](/assets/images/files/f7e44cac-1732-4bbc-ac1c-8405fa8a1d06.png "image")](/assets/images/files/be427703-7d74-4ad9-9ccf-949db8db04fd.png)<!--kg-card-end: html-->

Happy configuring!

