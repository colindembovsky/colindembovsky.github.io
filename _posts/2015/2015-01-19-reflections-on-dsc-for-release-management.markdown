---
layout: post
title: Reflections on DSC for Release Management
date: '2015-01-19 19:05:59'
tags:
- releasemanagement
---

A couple of months ago I did a series of posts ([this one](/using-webdeploy-in-vnext-releases) has the summary of all my RM/DSC posts) about using PowerShell DSC in Release Management. I set out to see if I could create a DSC script that RM could invoke that would prep the environment and install the application. I managed to get it going, but never felt particularly good about the final solution – it always felt a little bit hacky. Not the entire solution per se – really just the _application_ bit.

The main reason for this was the fact that I need to hack the Script Resource in order to let me run commands on the target node with parameters. Initially I thought that the inability to do this natively in DSC was short-sighted from the architecture of DSC – but the more I thought about it, the more I realized that I was trying to shoehorn application installation into DSC.

DSC scripts should be declarative – my scripts were mostly declarative, but the application-specific parts of the script were very much imperative – and that started to smell.

## Idempotency

I wrote about what I consider to be the most important mental shift when working with PowerShell DSC – idempotency. The scripts you create need to be idempotent – that is they need to end up in the same end state no matter what the starting state is. This works really well for the environment that an application needs to run in – but it doesn’t really work so well for the application itself.

My conclusion is simple: use DSC to specify the _environment_, and use plain ol’ PowerShell to install your _application_.

PowerShell DSC resources are split into 3 actions – Get, Test and Set. The Get method gets the state of the resource on the node. The Set method “makes it so” – it enforces the state the script specifies. The Test method checks to see if the target node’s state matches the state the script specifies. Let’s consider an example: the WindowsFeature resource. Consider the following excerpt:

    WindowsFeature WebServerRole
    {
        Name = "Web-Server"
        Ensure = "Present"
    }

When executing, this resource will check the corresponding WindowsFeature (IIS) on the target node using the Test method. If IIS is present, no action is taken (the node state matches the desired state specified in the script). If it’s not installed, the Set method is invoked to install/enable the IIS. Of course if we simply wanted to query the state of the WindowsFeature, the Get method would tell us the state (installed or not) of IIS.

This Get-Test-Set paradigm works well for environments – however, it starts to break down when you try to apply it to an application. Consider a Web Application with a SQL Database backend. How to you test if the application is in a particular state? You could check the schema of the database as an indication of the state; you could check if the site exists as an indication of the web site state. Of course this may not be sufficient for checking the state of your application.

(On a side note, if you’re using WebDeploy to deploy your website and you’re using Database Projects, you don’t need to worry, since these mechanisms are idempotent).

The point is, you may be deploying an application that doesn’t use an idempotent mechanism. In either case, you’re better off not trying to shoehorn application installation into DSC. Also, Release Management lets you execute both DSC and “plain” PowerShell against target nodes – so use them both.

## WebServerPreReqs Script

I also realized that I never published my “WebServerPreReqs” script. I use this script to prep a Web Server for my web application. There are four major sections to the script: Windows Features, runtimes, WebDeploy and MMC.

First, I ensure that Windows is in the state I need it to be – particularly IIS. I ensure that IIS is installed, as well as some other options like Windows authentication. Also, I ensure that the firewall allows WMI.

    Script AllowWMI 
    {
        GetScript = { @{ Name = "AllowWMI" } }
        TestScript = { $false }
        SetScript = 
        {
            Set-NetFirewallRule -DisplayGroup "Windows Management Instrumentation (WMI)" -Enabled True
        }
    }
    
    WindowsFeature WebServerRole
    {
        Name = "Web-Server"
        Ensure = "Present"
    }
    
    WindowsFeature WebMgmtConsole
    {
        Name = "Web-Mgmt-Console"
        Ensure = "Present"
        DependsOn = "[WindowsFeature]WebServerRole"
    }
    
    WindowsFeature WebAspNet
    {
        Name = "Web-Asp-Net"
        Ensure = "Present"
        DependsOn = "[WindowsFeature]WebServerRole"
    }
    
    WindowsFeature WebNetExt
    {
        Name = "Web-Net-Ext"
        Ensure = "Present"
        DependsOn = "[WindowsFeature]WebServerRole"
    }
    
    WindowsFeature WebAspNet45
    {
        Name = "Web-Asp-Net45"
        Ensure = "Present"
        DependsOn = "[WindowsFeature]WebServerRole"
    }
    
    WindowsFeature WebNetExt45
    {
        Name = "Web-Net-Ext45"
        Ensure = "Present"
        DependsOn = "[WindowsFeature]WebServerRole"
    }
    
    WindowsFeature WebHttpRedirect
    {
        Name = "Web-Http-Redirect"
        Ensure = "Present"
        DependsOn = "[WindowsFeature]WebServerRole"
    }
    
    WindowsFeature WebWinAuth
    {
        Name = "Web-Windows-Auth"
        Ensure = "Present"
        DependsOn = "[WindowsFeature]WebServerRole"
    }
    
    WindowsFeature WebScriptingTools
    {
        Name = "Web-Scripting-Tools"
        Ensure = "Present"
        DependsOn = "[WindowsFeature]WebServerRole"
    }

Next I install any runtimes my website requires – in this case, the MVC framework. You need to supply a network share somewhere for the installer – of course you could use a File resource as well, but you’d still need to have a source somewhere.

    #
    # Install MVC4
    #
    Package MVC4
    {
        Name = "Microsoft ASP.NET MVC 4 Runtime"
        Path = "$AssetPath\AspNetMVC4Setup.exe"
        Arguments = "/q"
        ProductId = ""
        Ensure = "Present"
        DependsOn = "[WindowsFeature]WebServerRole"
    }

I can’t advocate WebDeploy as a web deployment mechanism enough – if you’re not using it, you should be! However, in order to deploy an application remotely using WebDeploy, the WebDeploy agent needs to be running on the target node and the firewall port needs to be opened. No problem – easy to specify declaratively using DSC. I add the required arguments to get the installer to deploy and start the WebDeploy agent (see the Arguments setting in the Package WebDeploy resource). I also use a Script resource to Get-Test-Set the firewall rule for WebDeploy:

    #
    # Install webdeploy
    #
    Package WebDeploy
    {
        Name = "Microsoft Web Deploy 3.5"
        Path = "$AssetPath\WebDeploy_amd64_en-US.msi"
        Arguments = "ADDLOCAL=MSDeployFeature,MSDeployAgentFeature"
        ProductId = ""
        Ensure = "Present"
        Credential = $Credential
        DependsOn = "[WindowsFeature]WebServerRole"
    }
    
    #
    # Enable webdeploy in the firewall
    #
    Script WebDeployFwRule
    {
        GetScript = 
        {
            write-verbose "Checking WebDeploy Firewall exception status"
            $Rule = Get-NetFirewallRule -DisplayName "WebDeploy_TCP_8172"
            Return @{
                Result = "DisplayName = $($Rule.DisplayName); Enabled = $($Rule.Enabled)"
            }
        }
        SetScript =
        {
            write-verbose "Creating Firewall exception for WebDeploy"
            New-NetFirewallRule -DisplayName "WebDeploy_TCP_8172" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 8172
        }
        TestScript =
        {
            if (Get-NetFirewallRule -DisplayName "WebDeploy_TCP_8172" -ErrorAction SilentlyContinue) 
            {
                write-verbose "WebDeploy Firewall exception already exists"
                $true
            } 
            else 
            {
                write-verbose "WebDeploy Firewall exception does not exist"
                $false
            }
        }
        DependsOn = "[Package]WebDeploy"
    }

Finally, I wanted to make sure that MMC is installed so that I can monitor my application using Application Insights. This one was a little tricky since there isn’t an easy way to install the agent quietly – I have to unzip the installer and then invoke the MSI within. However, it’s still not that hard.

    #
    # MMA
    # Since this comes in an exe that can't be run silently, first copy the exe to the node,
    # then unpack it. Then use the Package Resource with custom args to install it from the
    # unpacked msi.
    #
    File CopyMMAExe
    {
        SourcePath = "$AssetPath\MMASetup-AMD64.exe"
        DestinationPath = "c:\temp\MMASetup-AMD64.exe"
        Force = $true
        Type = "File"
        Ensure = "Present"
    }
    
    Script UnpackMMAExe
    {
        DependsOn ="[File]CopyMMAExe"
        TestScript = { $false }
        GetScript = {
            @{
                Result = "UnpackMMAExe"
            }
        }
        SetScript = {
            Write-Verbose "Unpacking MMA.exe"
            $job = Start-Job { &amp; "c:\temp\MMASetup-AMD64.exe" /t:c:\temp\MMA /c }
            Wait-Job $job
            Receive-Job $job
        }
    }
    
    Package MMA
    {
        Name = "Microsoft Monitoring Agent"
        Path = "c:\temp\MMA\MOMAgent.msi"
        Arguments = "ACTION=INSTALL ADDLOCAL=MOMAgent,ACSAgent,APMAgent,AdvisorAgent AcceptEndUserLicenseAgreement=1 /qn /l*v c:\temp\MMA\mmaInstall.log"
        ProductId = ""
        Ensure = "Present"
        Dependson = "[Script]UnpackMMAExe"
    }

After running this script against a Windows Server in any state, I can be sure that the server will run my application – no need to guess or hope.

You can download the entire script from [here](http://1drv.ms/1IXntHL).

## Release Management

Now releasing my application is fairly easy in Release Management – execute two vNext script tasks: the first runs WebServerPreReqs DSC against the target node; the second runs a plain PowerShell script that invokes WebDeploy for my application using the drop folder of my build as the source.

## Conclusion

PowerShell DSC is meant to be declarative – any time you’re doing any imperative scripting, rip it out and put it into plain PowerShell. Typically this split is going to be along the line of _environment_ vs _application_. Use DSC for environment and plain PowerShell scripts for application deployment.

Happy releasing!

