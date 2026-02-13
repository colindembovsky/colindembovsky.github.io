---
layout: post
title: Tips and Tricks for Complex IaaS Deployments Using VSTS Deployment Groups
date: '2017-12-31 13:55:50'
tags:
- releasemanagement
- devops
---

Recently I was working with a customer that was struggling with test environments. Their environments are complex and take many weeks to provision and configure - so they are generally kept around even though some of them are not frequently used. Besides a laborious, error-prone manual install and configuration process that usually takes over 10 business days, the team has to maintain all the clones of this environment. This means that at least two senior team members are required just to maintain existing dev and test environments as well as create new ones.

Using [Azure ARM templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates) and VSTS [Release Management](https://docs.microsoft.com/en-us/vsts/build-release/concepts/definitions/release/what-is-release-management) with [Deployment Groups](https://docs.microsoft.com/en-us/vsts/build-release/concepts/definitions/release/deployment-groups/), we were able to show how we could spin up the entire environment in just under two hours. That's a 50x improvement in lead time! And it's more consistent since the entire process is automated and the scripts are all source controlled so there's auditability. This is a huge win for the team. Not only can they spin up an environment in a fraction of the time they are used to, they can now decommission environments that are not frequently used (some environments were only used twice a year). That means they have less maintenance to worry about. When they need an environment for a short time, they spin it up, used it and then throw it away. They've also disseminated "tribal knowledge" from a few team members' heads to a button click - meaning anyone can create a new environment now.

This was my first time working with a larger scale provisioning and configuration project that uses Deployment Groups - and this post documents some of the lessons that we learned along the way.

### A Brief Glossary

Before we jump into the tips, I need to get some definitions out of the way. In VSTS, a Release Definition is made up of multiple Environments. Typically you see DEV, STAGE and PROD but you can have multiple "environments" that target the same set of machines.

<!--kg-card-begin: html-->[![image](/assets/images/files/56561a03-700d-476e-9c63-7a6e90c7fa83.png "image")](/assets/images/files/d7bd08ff-5b27-47b4-b0f8-4aeb94295793.png)<!--kg-card-end: html-->

The above VSTS release has three "environments":

- Infrastructure Provision
- Runs an ARM template to provision VMs, VNets, Storage and any other infrastructure required for the environment
- Infrastructure Config
- Configure the OS of each machine, DNS and any other "low-level" settings
- App Install
- Install and configure the application(s)

This separation also allows you to run the "Infrastructure Provision" environment and then set it to a manual trigger and just trigger the config environment - particularly useful when you're developing the pipeline, since you can skip environments that end up being no-ops but take a couple minutes to pass through.

Within an Environment, you can have 1.._n_ phases. You specify tasks inside a phase - these are the smallest unit of work in the release.

<!--kg-card-begin: html-->[![image](/assets/images/files/50b0fd52-f99c-4a97-bbbf-2306512d0f68.png "image")](/assets/images/files/2aa9823e-f332-4cbe-be09-427257de1fb8.png)<!--kg-card-end: html-->

In the above image, there are several phases within the "Infrastructure Config" environment. Each phase (in this case) is running a single task, but you can run as many tasks as you need for that particular phase.

There are three types of phases: agentless, agent-based or deployment-group based. You can think of agentless phases as phases that are executed on VSTS. Agent-based phases are executed on agent(s) in a build or release queue. Deployment Group phases are executed on all agents (with optional tag matching) within the specified Deployment Group. The agent for agent-based or deployment-group based is the same agent under the hood - the difference is that deployment group agents are only referenced through the Deployment Group while build/release agents are accessed through queues. You'd typically use agent queues for build servers or for "proxy servers" in releases (where the tasks are executing on the proxy but acting on other machines). Deployment Groups are used when you don't know the machines ahead of time - like when you're spinning up a set of machines in the cloud on demand. They also allow you to target multiple machines at the same time.

The VSTS Deployment agent joins a machine (this can be any machine anywhere that can connect to VSTS) to a Deployment Group. The agent is cross-platform (runs on DotNET Core) so it can run on practically any machine anywhere. It connects out to VSTS meaning you don't need to open incoming firewall ports at all. The agent runs on the machine and so any scripts you write can execute locally - which simplifies configuration dramatically. Executing remote instructions is typically much harder to do - you have to think about your connection and security and so on. Executing locally is much easier.

## TL;DR - The Top 10 Tips and Tricks

Here are my top 10 tips and tricks:

1. Spin Up Azure VMs with the VSTS Deployment Agent Extension
2. This allows you to configure everything else locally on each machine
3. Use Tagging for Parallelization and Specialization
4. Tagging the VSTS agent allows you to repeat the same actions on many machines in parallel and/or distinguish machines for unique actions
5. Use Phases to Start New Sessions
6. Each phase in an Environment gets a new session, which is useful in a number of scenarios
7. Update Your PowerShell PackageProviders and Install DSC Modules
8. If you're using DSC, install the modules in a separate step to ensure that they are available when you run DSC scripts. You may need to update your Package Providers for this to work
9. Install Azure PowerShell and use the [Azure PowerShell task](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/AzurePowerShell)
10. If you're going to be doing any scripting to Azure, you can quickly install Azure PowerShell so that you can use the Azure PowerShell task
11. Use PowerShell DSC for OS Configuration
12. Configuring Windows Features, firewalls and so on is best done with PowerShell DSC
13. Use Plain PowerShell for Application Install and Config
14. Expressing application state can be challenging - so use "plain" PowerShell for application install and config
15. Attaching Data Disks in Azure VMs
16. If you add data disks in your ARM template, you still need to mount them in the OS of the VM
17. Configuring DNS on Azure VNets
18. If you create an Active Directory Domain Controller or DNS, you'll need to do some other actions on the VNet too
19. Wait on machines when they reboot
20. If you reboot a machine and don't pause, the subsequent deployment steps fail because the agent goes offline.

In the next section I'll dig into each tip.

### Tip 1: Spin Up Azure VMs with the VSTS Deployment Agent Extension

You can install the VSTS Deployment Agent (or just "the agent" for the remainder of this post) on any machine using a simple script. The script downloads the agent binary and configures it to connect the agent to your VSTS account and to the specified Deployment Group. However, if you're spinning up machines by using an ARM template, you can also install the agent via the VSTS extension. In order to do this you need a Personal Access Token (or PAT), the name of the VSTS account, the name of the Deployment Group and optionally some tags to tag the agent with. Tags will be important when you're distinguishing between machines in the same Deployment Group later on. You'll need to create the Deployment Group in VSTS before you run this step.

Here's a snippet of an ARM template that adds the extension to the Deployment Group:

    {
        "name": "[parameters('settings').vms[copyIndex()].name]",
        "type": "Microsoft.Compute/virtualMachines",
        "location": "[resourceGroup().location]",
        "apiVersion": "2017-03-30",
        "dependsOn": [
          ...
        ],
        "properties": {
          "hardwareProfile": {
            "vmSize": "[parameters('settings').vms[copyIndex()].size]"
          },
          "osProfile": {
            "computerName": "[parameters('settings').vms[copyIndex()].name]",
            "adminUsername": "[parameters('adminUsername')]",
            "adminPassword": "[parameters('adminPassword')]"
          },
          "storageProfile": {
            "imageReference": "[parameters('settings').vms[copyIndex()].imageReference]",
            "osDisk": {
              "createOption": "FromImage"
            },
            "dataDisks": [
                {
                    "lun": 0,
                    "name": "[concat(parameters('settings').vms[copyIndex()].name,'-datadisk1')]",
                    "createOption": "Attach",
                    "managedDisk": {
                        "id": "[resourceId('Microsoft.Compute/disks/', concat(parameters('settings').vms[copyIndex()].name,'-datadisk1'))]"
                    }
                }
            ]
          },
          "networkProfile": {
            "networkInterfaces": [
              {
                "id": "[resourceId('Microsoft.Network/networkInterfaces', concat(parameters('settings').vms[copyIndex()].name, if(equals(parameters('settings').vms[copyIndex()].name, 'JumpBox'), '-nicpub', '-nic')))]"
              }
            ]
          }
        },
        "resources": [
          {
            "name": "[concat(parameters('settings').vms[copyIndex()].name, '/TeamServicesAgent')]",
            "type": "Microsoft.Compute/virtualMachines/extensions",
            "location": "[resourceGroup().location]",
            "apiVersion": "2015-06-15",
            "dependsOn": [
              "[resourceId('Microsoft.Compute/virtualMachines/', concat(parameters('settings').vms[copyIndex()].name))]"
            ],
            "properties": {
              "publisher": "Microsoft.VisualStudio.Services",
              "type": "TeamServicesAgent",
              "typeHandlerVersion": "1.0",
              "autoUpgradeMinorVersion": true,
              "settings": {
                "VSTSAccountName": "[parameters('vstsAccount')]",
                "TeamProject": "[parameters('vstsTeamProject')]",
                "DeploymentGroup": "[parameters('vstsDeploymentGroup')]",
                "Tags": "[parameters('settings').vms[copyIndex()].tags]"
              },
              "protectedSettings": {
                "PATToken": "[parameters('vstsPat')]"
              }
            }
          },
          ...

Notes:

- The extension is defined in the highlighted lines
- The "settings" section of the extension is where you specify the VSTS account name, team project name, deployment group name and comma-separated list of tags for the agent. You also need to supply a PAT that has access to join machines to the Deployment Group
- You can also specify a "Name" property if you want the agent name to be custom. By default it will be machineName-DG (so if the machine name is WebServer, the agent will be named WebServer-DG.

Now you have a set of VMs that are bare-boned but have the VSTS agent installed. They are now ready for anything you want to throw at them - and you don't need to worry about ports or firewalls or anything like that.

There are some useful extensions and patterns for configuring VMs such as [Custom Script](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/extensions-customscript) or [Join Domain](https://azure.microsoft.com/en-us/resources/templates/201-vm-domain-join/). The problem with these scripts is that the link to the script has to be either in a public place or in a blob store somewhere, or they assume existing infrastructure. This can complicate deployment. Either you need to publish your scripts publically or you have to deal with uploading scripts and generating SAS tokens. So I recommend just installing the VSTS agent and let it do everything else that you need to do - especially since the agent will download artifacts (like scripts and build binaries) as a first step in any deployment phase.

### Tip 2: Use Tagging for Parallelization and Specialization

Tags are really important for Deployment Groups. They let you identify machines or groups of machines within a Deployment Group. Let's say you have a load balanced application with two webservers and a SQL server. You'd probably want identical configuration for the webservers and a completely different configuration for the SQL server. In this case, tag two machines with WEBSERVER and the other machine with SQL. Then you'll define the tasks in the phase &nbsp;- when the phase runs, it executes all the tasks on all the machines that match the filter - for example, you can target all WEBSERVER machines with a script to configure IIS. These will execute in parallel (you can configure it to work serially if you want to) and so you'll only specify the tasks a single time in the definition and you'll speed up the deployment.

<!--kg-card-begin: html-->[![image](/assets/images/files/5d918fca-64d5-4b20-a027-6afcb74dca7e.png "image")](/assets/images/files/2d415fda-4852-4327-adcd-07bd80234a0e.png)<!--kg-card-end: html-->

Be careful though: multiple tags use AND (not OR) logic. This means if you want to do something like join a domain on machines with WEBSERVER and SQL, you would think you could specify WEBSERVER, SQL as the tag filter in the phase tag filter. But since the tags are joined with an AND, you'll see the phase won't match any machines. So you'd have to add a NODE tag (or something similar) and apply it to both webservers and SQL machine and then target NODE for things you want to do on all the machines.

<!--kg-card-begin: html-->[![image](/assets/images/files/0858d323-3c20-4bf1-9211-1d90279e32df.png "image")](/assets/images/files/d5c254eb-a976-47a4-a9eb-7b375a8d080c.png)<!--kg-card-end: html-->

The above image shows the tag filtering on the Phase settings. Note too the parallelization settings.

### Tip 3: Use Phases to Start New Sessions

At my customer we were using Windows 2012 R2 machines. However, we wanted to use PowerShell DSC for configuring the VMs and you need Windows Management Framework 5.0 to get DSC. So we executed a PowerShell task to upgrade the PowerShell to 5.x:

    if ($PSVersionTable.PSVersion.Major -lt 5) {
        $powershell5Url = "https://go.microsoft.com/fwlink/?linkid=839516"
        wget -Uri $powershell5Url -OutFile "wmf51.msu"
        Start-Process .\wmf51.msu -ArgumentList '/quiet' -Wait
    }

Notes:

- Line 1: This script checks the major version of the current PowerShell
- Lines 2,3: If it's less than 5, then the script downloads PowerShell 5.1 (the path to the installer can be update to whichever PowerShell version you need)
- Line 4: The installer is invoked with the quiet parameter

However, if we then called a task right after the update task, we'd still get the old PowerShell since all tasks within a phase are executed in _the same session_. We just added another phase with the same Deployment Group settings - the second phase started a new session and we got the upgraded PowerShell.

This doesn't work for environment variables though. When you set machine environment variables, you have to restart the agent. The VSTS team are working on providing a task to do this, but for now you have to reboot the machine. We'll cover how to do this in Tip 10.

### Tip 4: Update Your PowerShell PackageProviders and Install DSC Modules

You really should be using PowerShell DSC to configure Windows. The notation is succinct and fairly easy to read and Windows plays nicely with DSC. However, if you're using custom modules (like xNetworking) you have to ensure that the modules are installed. You can pre-install all the modules so that your scripts can assume the modules are already installed. To install modules you'll need to update your Package Providers. Here's how to do it:

    Import-Module PackageManagement
    Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force

You'll need to start a new Phase in order to pick up the new packages. Then you'll be able to install modules:

    Install-Module -Name xActiveDirectory -Force
    Install-Module -Name xNetworking -Force
    Install-Module -Name xStorage -Force
    Install-Module -Name xDSCDomainjoin -Force
    Install-Module -Name xComputerManagement -Force

Not all machines need all the modules, but this step is so quick I found it easier to just enumerate and install all the modules anyway. That way I know that any machine could run any DSC script I throw at it.

### Tip 5: Install Azure PowerShell

If you're going to do anything against Azure from the VMs (in our case we were downloading binaries from a blob store) then you'll want to use the Azure PowerShell task. This task provides an authenticated context (via a preconfigured endpoint) so you don't have to worry about adding passwords or anything to your script. However, for it to work, you'll need to install Azure PowerShell. Again this must be a separate phase so that subsequent phases can make use of the Azure cmdlets. To do this simply add a PowerShell task and run this line of script:

<!--kg-card-begin: html--><font face="Courier New">Install-Module AzureRM -AllowClobber -Force</font><!--kg-card-end: html-->
### Tip 6: Use PowerShell DSC for OS Configuration

OS configuration can easily be specified by describing the state: is IIS installed or not? Which other OS roles are installed? So DSC is the perfect tool for this kind of work. You can use a single DSC script to configure a group of machines (or nodes, in DSC) but since we have the VSTS agent you can simply write your scripts for each machine using "node localhost". DSC script are also (usually) idempotent - so they work no matter what state the environment is in when the script executes. No messy if statements to check various conditions - DSC does it for you.

When you're doing DSC, you should first check if there is a "native" resource for your action - for example, configuring Windows Features uses the WindowsFeature resource. However, there are some custom actions you may want to perform. There are tons of extensions out there - we used xActiveDirectory to configure an Active Directory Domain Controller settings, for example.

There are times when you'll want to do some custom work that there simply is no custom module for. In that case, you'll need to use the Script resource. The script resource is composed of three parts: GetScript, TestScript and SetScript. GetScript is optional and should return the current state as an object if specified. TestScript should return a boolean - true for "the state is correct" or false for "the state is not correct". If TestScript returns a false, then the SetScript is invoked. Here's an example Script we wrote to configure SMB on a machine according to Security requirements:

    Script SMBConfig
    {
    	GetScript = { @{ Result = Get-SmbServerConfiguration } }
    	TestScript =
    	{
    		$config = Get-SmbServerConfiguration
    		$needConfig = $config.EnableSMB2Protocol -and (-not ($config.EnableSMB1Protocol))
    		if ($needConfig) {
    				Write-Host "SMB settings are not correct." 
    		}
    		$needConfig
    	}
    	SetScript =
    	{
    		Write-Host "Configuring SMB settings" 
    		Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force
    		Set-SmbServerConfiguration -EnableSMB2Protocol $true -Force
    	}
    }

Notes:

- Line 1: Specify the type of resource (Script) and a unique name
- Line 3: The GetScript returns an hash table with a Result property that describes the current state - in this case, the SMB settings on the machine
- Line 4: The start of the TestScript
- Line 6: Query the SMB settings
- Line 7: Determine if we need to configure anything or not - this is a check on the SMBProtocol states
- Lines 8-10: Write a message if we do need to set state
- Line 11: return the bool: true if the state is correct, false otherwise
- Lines 16-17: correct the state of the machine - in this case, set protocols accordingly

### Tip 7: Use Plain PowerShell for Application Install and Config

Expressing application state and configuration as a DSC script can be challenging. I once wrote some DSC that could [install SQL](/install-and-configure-sql-server-using-powershell-dsc). However, I ended up using a Script resource - and the TestScript just checked to see if a SQL service was running. This check isn't enough to determine if SQL features are installed according to some config.

Instead of writing long Script resources, I just revert to "plain" PowerShell for app install and configuration. This is especially true for more complicated apps. Just make sure your script are _idempotent_ - that is that they can run and succeed every time. For example, if you're installing a service, you may want to first check to see if the service exists before running the installer (otherwise the installer may fail since the service already exists). This allows you to re-run scripts again if other scripts fail.

### Tip 8: Attaching Data Disks in Azure VMs

If you're creating data disks for your VMs, then you usually specify the size and type of disk in the ARM template. But even if you add a disk, you need to attach it in the OS. To do this, I used the [xStorage DSC extension](https://github.com/PowerShell/xStorage). This requires a disk number. When we started, the data disk was always disk 2. Later, we added [Azure Disk Encryption](https://docs.microsoft.com/en-us/azure/security/azure-security-disk-encryption) - but this added another disk and so our disk numbers were off. We ended up needing to add some logic to determine the data disk number and pass that in as a parameter to the DSC configuration:

    Configuration DiskConfig
    {
    	param
    	(
    		[Parameter(Mandatory)]
    		[string]$dataDiskId
    	)
    
    	# import DSC Resources 
    	Import-DscResource -ModuleName PSDscResources
    	Import-DscResource -ModuleName xStorage
    
    	Node localhost
    	{
            LocalConfigurationManager
    		{
    			ActionAfterReboot = 'ContinueConfiguration'
    			ConfigurationMode = 'ApplyOnly'
    			RebootNodeIfNeeded = $true
    		}
    
            xWaitforDisk DataDisk
            {
                DiskId = $dataDiskId
                RetryIntervalSec = 60
                RetryCount = 3
            }
    
            xDisk FVolume
            {
                DiskId = $dataDiskId
                DriveLetter = 'F'
                FSLabel = 'Data'
                DependsOn = "[xWaitforDisk]DataDisk"
            }
        }
    }
    
    # work out what disk number the data disk is on
    $dataDisks = Get-Disk -FriendlyName "Microsoft Virtual Disk" -ErrorAction SilentlyContinue
    if ($dataDisk -eq $null) {
    	$dataDisk = Get-Disk -FriendlyName "Microsoft Storage Space Device" -ErrorAction SilentlyContinue
    }
    # filter to GPT partitions
    $diskNumber = 2
    Get-Disk | Out-Host -Verbose
    $dataDisk = Get-Disk | ? { $_.PartitionStyle -eq "RAW" -or $_.PartitionStyle -eq "GPT" }
    if ($dataDisk -eq $null) {
    	Write-Host "Cannot find any data disks"
    } else {
    	if ($dataDisk.GetType().Name -eq "Object[]") {
    		Write-Host "Multiple data disks"
    		$diskNumber = $dataDisk[0].Number
    	} else {
    		Write-Host "Found single data disk"
    		$diskNumber = $dataDisk.Number
    	}
    }
    Write-Host "Using $diskNumber for data disk mounting"
    
    DiskConfig -ConfigurationData .\ConfigurationData.psd1 -dataDiskId "$($diskNumber)"
    Start-DscConfiguration -Wait -Force -Path .\DiskConfig -Verbose 

Notes:

- Lines 3-7: We specify that the script requires a dataDiskId parameter
- Lines 11,12: Import the modules we need
- Lines 23-28: Wait for disk with number $dataDiskId to be available (usually it was immediately anyway)
- Lines 30-36: Mount the disk and assign drive letter F with a label of Data
- Lines 41-43: Get potential data disks
- Lines 46-59: Calculate the data disk number, defaulting to 2
- Lines 62,63: Compile the DSC and the invoke the configuration manager to "make it so"

### Tip 9: Configuring DNS on Azure VNets

In our example, we needed a Domain Controller to be on one of the machines. We were able to configure the domain controller using DSC. However, I couldn't get the other machines to join the domain since they could never find the controller. Eventually I realized the problem was a DNS problem. So we added the DNS role to the domain controller VM. We also added a private static IP address for the domain controller so that we could configure the VNet accordingly. Here's a snippet of the DSC script for this:

    WindowsFeature DNS
    {
            Ensure = "Present"
            Name = "DNS"
            DependsOn = "[xADDomain]ADDomain"
    }
    
    xDnsServerAddress DnsServerAddress
    {
            Address = '10.10.0.4', '127.0.0.1'
            InterfaceAlias = 'Ethernet 2'
            AddressFamily = 'IPv4'
            DependsOn = "[WindowsFeature]DNS"
    }

Notes:

- Lines 1-6: Configure the DNS feature
- Lines 8-14: Configure the network DNS NIC using the static private IP 10.10.0.4

Now we needed to configure the DNS on the Azure VNet to use the domain controller IP address. We used this script:

    param($rgName, $vnetName, $dnsAddress)
    $vnet = Get-AzureRmVirtualNetwork -ResourceGroupName $rgName -Name $vnetName
    if ($vnet.DhcpOptions.DnsServers[0] -ne $dnsAddress) {
        $vnet.DhcpOptions.DnsServers = @($dnsAddress)
        Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
    }

Notes:

- Line 2: Get the VNet using the resource group name and VNet name
- Line 3: Check if the DNS setting of the VNet is correct
- Lines 4,5: If it's not, then set it to the internal IP address of the DNS server

This script needs to run as an [Azure PowerShell script task](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/AzurePowerShell) so that it's already logged in to an Azure context (the equivalent of running Login-AzureRMAccount -ServicePrincipal). It's sweet that you don't have to provide any credentials in the script!

Now that we've set the DNS on the VNet, we have to reboot every machine on the VNet (otherwise they won't pick up the change). That brings us to the final tip.

### Tip 10: Wait on Machines When They Reboot

You can easily reboot a machine by running this (plain) PowerShell:

<!--kg-card-begin: html--><font face="Courier New">Restart-Machine -ComputerName localhost -Force</font><!--kg-card-end: html-->

. This is so simple that you can do it as an inline PowerShell task:

<!--kg-card-begin: html-->[![image](/assets/images/files/a2dd904d-386d-4f9a-8a47-a200a8aba5e7.png "image")](/assets/images/files/c7a4e385-8587-4347-a7a5-77aff6f7bb7b.png)<!--kg-card-end: html-->

Rebooting the machine is easy: it's waiting for it to start up again that's more challenging. If you have a task right after the reboot task, the deployment fails since the agent goes offline. So you have to build in a wait. The simplest method is to add an agentless phase and add a Delay task:

<!--kg-card-begin: html-->[![image](/assets/images/files/be0c7bbe-79ed-4cb1-85e6-ed9b61d7ca42.png "image")](/assets/images/files/92f15bf4-6607-4aff-85d7-a1d1a102cec2.png)<!--kg-card-end: html-->

However, you can be slightly more intelligent if you poll the machine states using some Azure PowerShell:

    param (
        [string]$ResourceGroupName,
        [string[]]$VMNames = @(),
        $TimeoutMinutes = 2,
        $DelaySeconds = 30
    )
    
    Write-Host "Delay for $DelaySeconds seconds."
    Start-Sleep -Seconds $DelaySeconds
    
    if($VMNames.Count -eq 0)
    {
        $VMNames = (Get-AzureRmVm -ResourceGroupName $ResourceGroupName).Name
        Write-Host "Getting VM names."
    }
    
    $seconds = 10
    $desiredStatus = "PowerState/running"
    
    foreach($vmName in $VMNames)
    {
        $timer = [Diagnostics.Stopwatch]::StartNew()
        Write-Host "Getting statuses of VMs."
        $statuses = (Get-AzureRmVm -ResourceGroupName $ResourceGroupName -VMName $vmName -Status).Statuses
        $status = $statuses | Where-Object { $_.Code -eq $desiredStatus }
        while($status -eq $null -and ($timer.Elapsed.TotalMinutes -lt $TimeoutMinutes))
        {
            Write-Verbose "Retrying in $($seconds) seconds."
            Start-Sleep -Seconds $seconds
            $statuses = (Get-AzureRmVm -ResourceGroupName $ResourceGroupName -VMName $vmName -Status).Statuses
            $status = $statuses | Where-Object { $_.Code -eq $desiredStatus }
        }
    
        if($timer.Elapsed.TotalMinutes -ge $TimeoutMinutes)
        {
            Write-Error "VM restart exceeded timeout."
        }
        else
        {
            Write-Host "VM name $($vmName) has current status of $($status.DisplayStatus)."
        }
    }

Notes:

- The script requires a resource group name, an optional array of machine names (otherwise it will poll all the VMs in the resource group), a delay (defaulted to 30 seconds) and a timeout (defaulted to 2 minutes)
- The script will delay for a small period (to give the machines time to start rebooting) and then poll them until they're all running or the timeout is reached.

This script has to run in an Azure PowerShell task in an _agent_ phase from either the Hosted agent or a private agent - you can't run it in a Deployment Group phase since those machines are rebooting!

## Conclusion

Deployment Groups are very powerful - they allow you to dynamically target multiple machines and execute configuration in a local context. This makes complex environment provisioning and configuration much easier to manage. However, it's always good to know limitations, gotchas and practical tips when designing a complex deployment workflow. Hopefully these tips and tricks make your life a bit easier.

Happy deploying!

