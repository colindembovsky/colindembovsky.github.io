---
layout: post
title: Error Handling Poor Man’s RunScript in 2012 Builds
date: '2014-01-23 16:45:00'
tags:
- build
---

Yesterday I posted about [how to create script hooks in a 2012 build template](http://www.colinsalmcorner.com/2014/01/build-script-hooks-for-tfs-2012-builds.html). My colleague Tyler Doerksen commented and pointed out that there was no error handling in my solution.

## Returning Error Codes from PowerShell

I knew that if I could get the script to return an error code, it would be simple to add an If activity to check it. The trick is to use “exit 0” for success and “exit 1” for failures. I also changed any error messages from Write-Host to Write-Error so that they go to errOutput and not stdOutput. Here’s the updated “UpdateVersion” script:

    Param(
      [string]$pathToSearch = $env:TF_BUILD_SOURCESDIRECTORY,
      [string]$buildNumber = $env:TF_BUILD_BUILDNUMBER,
      [string]$searchFilter = "AssemblyInfo.*",
      [regex]$pattern = "\d+\.\d+\.\d+\.\d+"
    )
    
    if ($buildNumber -match $pattern -ne $true) {
        Write-Error "Could not extract a version from [$buildNumber] using pattern [$pattern]"
        exit 1
    } else {
        try {
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
            exit 0
        } catch {
            Write-Error $_
            exit 1
        }
    }

## Error Handling in the Build

Go back to the InvokeProcess activity that calls your script. Go to its parent activity (usually a sequence) and add a variable Int32 called “scriptResult”. On the InvokeProcess, set the result property to “scriptResult”.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-4tYWfCn9V_I/UuC6f6pcTPI/AAAAAAAABNU/_SeZD3uAPiM/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-qNekJ3Hour8/UuC6fEQ-p7I/AAAAAAAABNM/M4gYUr9O4x4/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

Now you just need to add an If activity below the InvokeProcess that has condition “scriptResult \<\> 0” and add a Throw in the “Then”. I’m just throwing an Exception with an error message.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-ozo4fmKIliU/UuC6hZtLoEI/AAAAAAAABNk/qT4IrldRB30/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-MTQdOTgLRDg/UuC6gsK3_II/AAAAAAAABNc/qC-XTk2fIWg/s1600-h/image%25255B7%25255D.png)<!--kg-card-end: html-->

Here’s the output if the script fails:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-Q89AqukHG8E/UuC6jJdpQlI/AAAAAAAABN0/vvy9_ZMnmF8/image_thumb%25255B5%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/--a1xaEwoCgI/UuC6iVATnlI/AAAAAAAABNs/-ixIhLWldus/s1600-h/image%25255B11%25255D.png)<!--kg-card-end: html-->

Happy building!

