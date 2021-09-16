---
layout: post
title: Enabling JavaScript Code Coverage Link in Builds
date: '2014-05-14 20:43:03'
tags:
- build
---

In a [previous post](/unit-testing-javascript-in-vs-2012) I wrote about how to do JavaScript unit testing in VS 2012. The same procedure applies to VS 2013 – but the [Chutzpah test adapter](http://visualstudiogallery.msdn.microsoft.com/f8741f04-bae4-4900-81c7-7c9bfb9ed1fe) now allows you to run code coverage too. At the end of my post I link to another blog entry about how to enable the tests to run during team builds.

I recently added some tests to a VS 2013 solution I was working on and was pleased to see that when you “Analyze Code Coverage for all Tests” in the Test Explorer, VS pops open a nicely formatted html page that shows you your JavaScript coverage. I wanted to have that file available in my build results too. Looking at the test results folder of the local VS test run, I saw that Chutzpah created an html file called “\_Chutzpah.coverage.html”. I wanted that script to be copied to the drop folder of the build and create a link in the build summary that you could click to open it.

## Post-Test Script

Fortunately you can do this without even having to customize the build template – as long as you’re using the TfvcTemplate.12.xaml template – the default build template that ships with TFS 2013. This build has some really useful script hooks – and there’s one for running a post-test script. I knew I could easily copy the Chutpah result file to the drop folder – no problem. But how do you add to the build summary report from a script that’s running “outside” the workflow? If you customize the workflow you can use the WriteCustomSummaryInformation activity, but I wanted to do this without modifying the template.

After mailing the ChampsList, [Jakob Ehn](http://www.geekswithblogs.net/jakob/Default.aspx) pointed me in the right direction – I needed to use the [InformationNodeCoverters.AddCustomSummaryInformation](http://msdn.microsoft.com/en-us/library/microsoft.teamfoundation.build.client.informationnodeconverters.addcustomsummaryinformation.aspx) method. Once I had that, the rest of the PowerShell script was almost trivial. I did hit one snag – I need the Team Project Collection URI for the script to work, but for some reason the value of the TF\_BUILD\_COLLECTIONURI [build environment variable](http://msdn.microsoft.com/en-us/library/hh850448.aspx) was empty. Updating my build agent to VS 2013 Update 2 resolved this issue. Here’s the script:

    Param(
      [string]$testResultsDir = $env:TF_BUILD_TESTRESULTSDIRECTORY,
      [string]$dropLocation = $env:TF_BUILD_DROPLOCATION,
      [string]$tpcUri = $env:TF_BUILD_COLLECTIONURI,
      [string]$buildUri = $env:TF_BUILD_BUILDURI
    )
    
    $coverageFileName = "\_Chutzpah.coverage.html"
    $jsScriptResultsFile = $testResultsDir + $coverageFileName
    if (Test-Path($jsScriptResultsFile)) {
        try {
            Write-Host "Copying Chutzpah coverage files"
            copy $jsScriptResultsFile $dropLocation
    
            # add the link into the build summary
            Write-Host "Loading TFS assemblies"
            [Reflection.Assembly]::LoadWithPartialName('Microsoft.TeamFoundation.Client')
            [Reflection.Assembly]::LoadWithPartialName('Microsoft.TeamFoundation.Build.Client')
        
            Write-Host "Getting build object"
            $tpc = [Microsoft.TeamFoundation.Client.TfsTeamProjectCollectionFactory]::GetTeamProjectCollection($tpcUri)
            $buildService = $tpc.GetService([Microsoft.TeamFoundation.Build.Client.IBuildServer])
            $build = $buildService.GetBuild($buildUri)
    
            Write-Host "Writing Chutzpah coverage link to build summary"
            $message = "Javascript testing was detected. Open [coverage results]($dropLocation\$coverageFileName)"
            [Microsoft.TeamFoundation.Build.Client.InformationNodeConverters]::AddCustomSummaryInformation($build.Information, $message, "ConfigurationSummary", "Javascript Coverage", 200)
            $build.Information.Save();
    
            # all is well with the world
            Write-Host "Success!"
            exit 0
        }
        catch {
            Write-Error $_
            exit 1
        }
    } else {
        # let the build know there were no coverage files
        Write-Warning "No Chutzpah coverage file detected"
        exit 0
    }

I saved this to my build scripts folder under source control and checked it in.

Opening up the build definition, I had to create a second build run to run the JavaScript tests – here’s the settings I used:

<!--kg-card-begin: html-->[![image](/assets/images/files/b236a53c-1ed2-4625-b4b9-340b571d9299.png "image")](/assets/images/files/a4f94785-d8fd-4e7b-af54-c648bc1c77a6.png)<!--kg-card-end: html-->

Note how I’ve enabled Code Coverage in the options dropdown.

I added this folder to the source mappings for my build and then called the script in the post-test settings of the build:

<!--kg-card-begin: html-->[![image](/assets/images/files/c9497e2e-6641-4866-86ad-7bf08f41ba7e.png "image")](/assets/images/files/6ad64dfc-ef5a-4c5b-b343-f0a25e2a5071.png)<!--kg-card-end: html-->

Now when I run my build, I get a link to the JavaScript coverage file:

<!--kg-card-begin: html-->[![image](/assets/images/files/94d99bff-5c69-4239-a145-6795dd3916e4.png "image")](/assets/images/files/99bcacbb-0bc3-4cac-a6ac-980b849166a4.png)<!--kg-card-end: html-->

Clicking on the “coverage results” link opens the results page:

<!--kg-card-begin: html-->[![image](/assets/images/files/033ce68c-1531-47b8-8241-cc884562de5f.png "image")](/assets/images/files/e710bde6-a4bc-492b-a1df-d287ef0328b7.png)<!--kg-card-end: html-->

As a next project, I want to see if I can incorporate the coverage results into the build warehouse so that there’s metrics not only on .NET coverage over time, but also for JavaScript tests.

Happy testing!

