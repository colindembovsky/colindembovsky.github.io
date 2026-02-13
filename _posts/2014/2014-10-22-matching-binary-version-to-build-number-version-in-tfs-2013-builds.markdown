---
layout: post
title: Matching Binary Version to Build Number Version in TFS 2013 Builds
date: '2014-10-22 08:40:02'
tags:
- build
---

Jim Lamb wrote a [post](http://blogs.msdn.com/b/jimlamb/archive/2009/11/18/how-to-create-a-custom-workflow-activity-for-tfs-build-2010.aspx) about how to use a custom activity to match the compiled versions of your assemblies to the TFS build number. This was not a trivial exercise (since you have to edit the workflow itself) but is the best solution for this sort of operation. Interestingly the post was written in November 2009 and updated for TFS 2010 RTM in February 2010.

I finally got a chance to play with a VM that’s got [TFS 2013](http://www.microsoft.com/visualstudio/eng/2013-downloads) Preview installed. I was looking at the changes to the build engine. The Product Team have simplified the default template (they’ve collapsed a lot of granular activities into 5 or 6 larger activities). In fact, if you use the default build template, you won’t even see it (it’s not in the BuildProcessTemplates folder – you have to download it if you want to customize it).

The good news is that the team have added pre- and post-build and pre- and post-test script hooks into the default workflow. I instantly realised this could be used to solve the assembly-version-matches-build-number problem in a much easier manner.

## Using the Pre-Build Script

The solution is to use a PowerShell script that can replace the version in the AssemblyInfo files before compiling with the version number in the build. Here’s the procedure:

1. Import the UpdateVersion.ps1 script into source control (the script is below)
2. Change the build number format of your builds to produce something that contains a version number
3. Point the pre-build script argument to the source control path of the script in step 1

The script itself is pretty simple – find all the matching files (AssemblyInfo.\* by default) in a target folder (the source folder by default). Then extract the version number from the build number using a regex pattern, and do a regex replace on all the matching files.

If you’re using TFVC, the files are marked read-only when the build agent does a Get Latest, so I had to remove the read-only bit as well. The other trick was getting the source path and the build number – but you can use environment variables when executing any of the pre- or post- scripts (as detailed [here](http://msdn.microsoft.com/en-us/library/vstudio/dd647547(v=vs.120).aspx#scripts)).

    Param(
      [string]$pathToSearch = $env:TF_BUILD_SOURCESDIRECTORY,
      [string]$buildNumber = $env:TF_BUILD_BUILDNUMBER,
      [string]$searchFilter = "AssemblyInfo.*",
      [regex]$pattern = "\d+\.\d+\.\d+\.\d+"
    )
    
    try
    {
        if ($buildNumber -match $pattern -ne $true) {
            Write-Host "Could not extract a version from [$buildNumber] using pattern [$pattern]"
            exit 1
        } else {
            $extractedBuildNumber = $Matches[0]
            Write-Host "Using version $extractedBuildNumber"
    
            gci -Path $pathToSearch -Filter $searchFilter -Recurse | %{
                Write-Host " -&gt; Changing $($_.FullName)" 
            
                # remove the read-only bit on the file
                sp $_.FullName IsReadOnly $false
    
                # run the regex replace
                (gc $_.FullName) | % { $_ -replace $pattern, $extractedBuildNumber } | sc $_.FullName
            }
    
            Write-Host "Done!"
        }
    }
    catch {
        Write-Host $_
        exit 1
    }

Save this [script](http://sdrv.ms/1bbZKWJ) as “UpdateVersion.ps1” and put it into Source Control (I use a folder called $/Project/BuildProcessTemplates/CommonScripts to house all the scripts like this one for my Team Project).

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-7FXKUl0DzNA/Ue533vEyvuI/AAAAAAAAA-s/J1MR-03AQ5Y/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-pBXCKbcpwtQ/Ue532irjf6I/AAAAAAAAA-k/UXknumZQf9g/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

The open your build and specify the source control path to the pre-build script (leave the arguments empty, since they’re all defaulted) and add a version number to your build number format. Don’t forget to add the script’s containing folder as a folder mapping in the Source Settings tab of your build.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-WQFam_YQoo8/Ue535YZOfUI/AAAAAAAAA-8/58QHGMD0dY0/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-7VO2ED_AQM4/Ue534-SSWVI/AAAAAAAAA-0/cc_ZmHC4hu8/s1600-h/image%25255B7%25255D.png)<!--kg-card-end: html-->

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-ycoNgswTTeM/Ue536lSJTJI/AAAAAAAAA_M/zRdgbu0bNDQ/image_thumb%25255B5%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-oAFMEiiaeCI/Ue536Ou5d6I/AAAAAAAAA_E/1adNWaDcj28/s1600-h/image%25255B11%25255D.png)<!--kg-card-end: html-->

Now you can run your build, and your assembly (and exe) versions will match the build number:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-3MfeHizdJro/Ue5387D_kXI/AAAAAAAAA_c/WAq0aIxYQrI/image_thumb%25255B7%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-fWNmnsSowyI/Ue537sUflYI/AAAAAAAAA_U/fF85Zqp6eDo/s1600-h/image%25255B15%25255D.png)<!--kg-card-end: html-->

I’ve tested this script using TFVC as well as a TF Git repository, and both work perfectly.

Happy versioning!

