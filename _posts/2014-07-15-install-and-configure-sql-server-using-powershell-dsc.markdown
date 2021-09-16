---
layout: post
title: Install and Configure SQL Server using PowerShell DSC
date: '2014-07-15 21:35:13'
tags:
- devops
---

I’m well into my journey of discovering the capabilities of PowerShell DSC and Release Management’s DSC feature (See my previous posts: [PowerShell DSC: Configuring a Remote Node to “Reboot If Needed”](/powershell-dsc-remotely-configuring-a-node-to-rebootnodeifneeded), [Using PowerShell DSC in Release Management: The Hidden Manual](/using-powershell-dsc-in-release-management-the-hidden-manual) and [More DSC Release Management Goodness: Readying a Webserver for Deployment](/more-dsc-release-management-goodness-readying-a-webserver-for-deployment)). I’ve managed to work out how to use Release Management to run DSC scripts on nodes. Now I am trying to construct a couple of scripts that I can use to deploy applications to servers – including, of course, configuring the servers – using DSC. (All scripts for this post are available for download [here](http://1drv.ms/1n6C0J9)).

## SQL Server Installation

To install SQL Server via a script, there are two prerequisites: the SQL install sources and a silent (or unattended) installation command.

Fortunately the SQL server installer takes care of the install command – you run the install wizard manually, specifying your installation options as you go. On the last page, just before clicking “Install”, you’ll see a path to the ini conifguration file. I saved the configuration file and cancelled the install. Then I opened the config file and tweaked it slightly (see [this post](http://msdn.microsoft.com/en-us/library/dd239405.aspx) and [this post](http://www.mssqltips.com/sqlservertip/2511/standardize-sql-server-installations-with-configuration-files/) on some tweaking ideas)– till I could run the installer from the command line (using the /configurationFile switch). That takes care of the install command itself.

<!--kg-card-begin: html-->[![image](/assets/images/files/8eb3ba64-e66c-496f-8cdc-56ff047923ed.png "image")](/assets/images/files/b2b82361-a674-42ca-a2c3-f221f092fe92.png)<!--kg-card-end: html-->

There are many ways to make the SQL installation sources available to the target node. I chose to copy the ISO to the node (using the File DSC resource) from a network share, and then use a Script resource to mount the iso. Once it’s mounted, I can run the setup command using the ini file.

SQL Server requires .NET 3.5 to be installed on the target node, so I’ve added that into the script using the WindowsFeature resource. Here’s the final script:

    Configuration SQLInstall
    {
        param (
            [Parameter(Mandatory=$true)]
            [ValidateNotNullOrEmpty()]
            [String]
            $PackagePath,
    
            [Parameter(Mandatory=$true)]
            [ValidateNotNullOrEmpty()]
            [String]
            $WinSources
        )
    
        Node $AllNodes.where{ $_.Role.Contains("SqlServer") }.NodeName
        {
            Log ParamLog
            {
                Message = "Running SQLInstall. PackagePath = $PackagePath"
            }
    
            WindowsFeature NetFramework35Core
            {
                Name = "NET-Framework-Core"
                Ensure = "Present"
                Source = $WinSources
            }
    
            WindowsFeature NetFramework45Core
            {
                Name = "NET-Framework-45-Core"
                Ensure = "Present"
                Source = $WinSources
            }
    
            # copy the sqlserver iso
            File SQLServerIso
            {
                SourcePath = "$PackagePath\en_sql_server_2012_developer_edition_x86_x64_dvd_813280.iso"
                DestinationPath = "c:\temp\SQLServer.iso"
                Type = "File"
                Ensure = "Present"
            }
    
            # copy the ini file to the temp folder
            File SQLServerIniFile
            {
                SourcePath = "$PackagePath\ConfigurationFile.ini"
                DestinationPath = "c:\temp"
                Type = "File"
                Ensure = "Present"
                DependsOn = "[File]SQLServerIso"
            }
    
            #
            # Install SqlServer using ini file
            #
            Script InstallSQLServer
            {
                GetScript = 
                {
                    $sqlInstances = gwmi win32_service -computerName localhost | ? { $_.Name -match "mssql*" -and $_.PathName -match "sqlservr.exe" } | % { $_.Caption }
                    $res = $sqlInstances -ne $null -and $sqlInstances -gt 0
                    $vals = @{ 
                        Installed = $res; 
                        InstanceCount = $sqlInstances.count 
                    }
                    $vals
                }
                SetScript = 
                {
                    # mount the iso
                    $setupDriveLetter = (Mount-DiskImage -ImagePath c:\temp\SQLServer.iso -PassThru | Get-Volume).DriveLetter + ":"
                    if ($setupDriveLetter -eq $null) {
                        throw "Could not mount SQL install iso"
                    }
                    Write-Verbose "Drive letter for iso is: $setupDriveLetter"
                    
                    # run the installer using the ini file
                    $cmd = "$setupDriveLetter\Setup.exe /ConfigurationFile=c:\temp\ConfigurationFile.ini /SQLSVCPASSWORD=P2ssw0rd /AGTSVCPASSWORD=P2ssw0rd /SAPWD=P2ssw0rd"
                    Write-Verbose "Running SQL Install - check %programfiles%\Microsoft SQL Server\120\Setup Bootstrap\Log\ for logs..."
                    Invoke-Expression $cmd | Write-Verbose
                }
                TestScript =
                {
                    $sqlInstances = gwmi win32_service -computerName localhost | ? { $_.Name -match "mssql*" -and $_.PathName -match "sqlservr.exe" } | % { $_.Caption }
                    $res = $sqlInstances -ne $null -and $sqlInstances -gt 0
                    if ($res) {
                        Write-Verbose "SQL Server is already installed"
                    } else {
                        Write-Verbose "SQL Server is not installed"
                    }
                    $res
                }
            }
        }
    }
    
    # command for RM
    #SQLInstall -ConfigurationData $configData -PackagePath "\\rmserver\Assets" -WinSources "d:\sources\sxs"
    
    # test from command line
    SQLInstall -ConfigurationData configData.psd1 -PackagePath "\\rmserver\Assets" -WinSources "d:\sources\sxs"
    Start-DscConfiguration -Path .\SQLInstall -Verbose -Wait -Force

Here’s some analysis:

- (Line 7 / 12) The config takes in 2 parameters: $PackagePath (location of SQL ISO and config ini file) and $WinSources (Path to windows sources).
- (Line 15) I changed my config data so that I can specify a comma-separated list of roles (since a node might be a SQLServer and a WebServer) so I’ve made the comparer a “contains” rather than an equals (as I’ve had in my previous scripts) – see the config script below.
- (Line 22 / 29) Configure .NET 3.5 and .NET 4.5 Windows features, using the $WinSources path if the sources are required
- (Line 37) Copy the SQL iso to the target node from the $PackagePath folder
- (Line 46) Copy the ini file to the target node from the $PackagePath folder
- (Line 58) Begins the Script to install SQL server
- The Get-Script does a check to see if there is a SQL server service running. If there is, it returns the SQL instance count for the machine.
- The Set-Script mounts the iso, saving the drive letter to a variable. Then I invoke the setup script (passing in the config file and required passwords) writing the output to Write-Verbose, which will appear on the DSC invoking machine as the script executes.
- The Test-Script does the same basic “is there a SQL server service running” check. If there is, skip the install – else run the install. Of course this could be refined to ensure each and every component is installed, but I didn’t want to get that granular.
- The last couple of lines of the script show the command for Release Management (commented out) as well as the command to run the script manually from a PowerShell prompt.

Here’s my DSC config script:

    #$configData = @{
    @{
        AllNodes = @(
            @{
                NodeName = "*"
                PSDscAllowPlainTextPassword = $true
             },
    
            @{
                NodeName = "fabfiberserver"
                Role = "WebServer,SqlServer"
             }
        );
    }
    
    # Note: different 1st line for RM or command line invocation
    # use $configData = @{ for RM
    # use @{ for running from command line

You can download the above scripts (and my SQL configuration ini file for reference) [here](http://1drv.ms/1n6C0J9).

## What’s Next

After running this script, I have a server with SQL Server installed and configured according to my preferences (which are contained in the ini file). From here, I can run restores or dacpac deployments and so on. Of course this is going to be executed from within Release Management as part of the release pipeline.

Next up will be the full WebServer DSC script – and then we’ll be ready to tackle the actual application deployment, since we’ll have servers ready to host our applications.

Until then, happy releasing!

