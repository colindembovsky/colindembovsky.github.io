---
layout: post
title: 'Build vNext and SonarQube Runner: Dynamic Version Script'
date: '2015-08-11 19:34:58'
tags:
- build
- development
---

[SonarQube](http://www.sonarqube.org/) is a fantastic tool for tracking technical debt, and it’s starting to make some inroads into the .NET world as SonarSource [collaborates with Microsoft](http://www.sonarqube.org/announcing-sonarqube-integration-with-msbuild-and-team-build/). I’ve played around with it a little to start getting my hands dirty.

## Install Guidance

If you’ve never installed SonarQube before, then I highly recommend this [eGuide](https://github.com/SonarSource/sonar-.net-documentation). Just one caveat that wasn’t too clear: you need to create the database manually before running SonarQube for the first time. Just create an empty database (with the required collation) and go from there.

## Integrating into TeamBuild vNext – with Dynamic Versioning

Once you’ve got the server installed and configured, you’re ready to integrate with TeamBuild. It’s easy enough using [build VNext Command Line task](https://github.com/SonarSource/sonar-.net-documentation/blob/master/doc/analyze-from-tfs.md#analyzing-projects-using-the-new-tfs-2015-build-system). However, one thing bugged me as I was setting this up – hard-coding the version number. I like to [version my assemblies from the build number](/matching-binary-version-to-build-number-version-in-tfs-2013-builds) on the build using a PowerShell script. Here’s the 2015 version (since the [environment variable names](https://msdn.microsoft.com/Library/vs/alm/Build/scripts/variables) have changed):

    Param(
      [string]$pathToSearch = $env:BUILD_SOURCESDIRECTORY,
      [string]$buildNumber = $env:BUILD_BUILDNUMBER,
      [string]$searchFilter = "AssemblyInfo.*",
      [regex]$pattern = "\d+\.\d+\.\d+\.\d+"
    )
     
    if ($buildNumber -match $pattern -ne $true) {
        Write-Error "Could not extract a version from [$buildNumber] using pattern [$pattern]"
        exit 1
    } else {
        try {
            $extractedBuildNumber = $Matches[0]
            Write-Host "Using version $extractedBuildNumber in folder $pathToSearch"
     
            $files = gci -Path $pathToSearch -Filter $searchFilter -Recurse
    
            if ($files){
                $files | % {
                    $fileToChange = $_.FullName  
                    Write-Host " -&gt; Changing $($fileToChange)"
                    
                    # remove the read-only bit on the file
                    sp $fileToChange IsReadOnly $false
     
                    # run the regex replace
                    (gc $fileToChange) | % { $_ -replace $pattern, $extractedBuildNumber } | sc $fileToChange
                }
            } else {
                Write-Warning "No files found"
            }
     
            Write-Host "Done!"
            exit 0
        } catch {
            Write-Error $_
            exit 1
        }
    }

So now that I get dll’s versions matching my build number, why not SonarQube too? So I used the same idea to wrap the “begin” call into a PowerShell script which can get the build number too:

    Param(
      [string]$buildNumber = $env:BUILD_BUILDNUMBER,
      [regex]$pattern = "\d+\.\d+\.\d+\.\d+",
      [string]$key,
      [string]$name
    )
     
    $version = "1.0"
    if ($buildNumber -match $pattern -ne $true) {
        Write-Verbose "Could not extract a version from [$buildNumber] using pattern [$pattern]" -Verbose
    } else {
        $version = $Matches[0]
    }
    
    Write-Verbose "Using args: begin /v:$version /k:$key /n:$name" -Verbose
    $cmd = "MSBuild.SonarQube.Runner.exe"
    
    &amp; $cmd begin /v:$version /k:$key /n:$name

I drop this into the same folder as the MsBuild.SonarQube.Runner.exe so that I don’t have to fiddle with more paths. Here’s the task in my build:

<!--kg-card-begin: html-->[![image](/assets/images/files/54dbdda6-9975-48dd-875c-858fa7c55b0d.png "image")](/assets/images/files/8f4dad3d-6859-4220-a674-89b99f03542b.png)<!--kg-card-end: html-->

The call to the SonarQube runner “end” doesn’t need any arguments, so I’ve left that as a plain command line call:

<!--kg-card-begin: html-->[![image](/assets/images/files/995b513c-35c3-4919-be0b-e129c25cd40e.png "image")](/assets/images/files/1173de53-ef4d-4ea9-bc36-04d29ffd8f9a.png)<!--kg-card-end: html-->

Now when the build runs, the version number passed to SonarQube matches the version number of my assemblies which I can tie back to my builds. Sweet!

<!--kg-card-begin: html--> [![image](/assets/images/files/11a02d4c-7fd8-418c-9d81-1404f74f2ee4.png "image")](/assets/images/files/7f59f375-b594-4b3a-bee7-adff7b134e47.png)<!--kg-card-end: html-->

One more change you could make is to specify the key and name arguments as variables. That way you can manage them as build variables instead of managing them in the call to the script on the task.

Finally, don’t forget to install the Roslyn SonarQube [SonarLint extension](https://visualstudiogallery.msdn.microsoft.com/47d1049d-bb27-454e-aab8-24566c85e548?SRC=Home). This will give you the same analysis that SonarQube uses inside VS.

Happy SonarQubing!

