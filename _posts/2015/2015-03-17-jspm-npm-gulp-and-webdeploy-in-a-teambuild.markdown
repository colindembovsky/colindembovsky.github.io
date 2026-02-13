---
layout: post
title: JSPM, NPM, Gulp and WebDeploy in a TeamBuild
date: '2015-03-17 17:00:27'
tags:
- development
---

I’ve been coding a web project using [Aurelia](http://aurelia.io) for the last couple of weeks (more posts about what I’m actually doing to follow soon!). Aurelia is an amazing SPA framework invented by Rob Eisenberg ([@EisenbergEffect](https://twitter.com/eisenbergeffect)).

## JSPM

Aurelia utilizes [npm (Node Package Manager)](https://www.npmjs.com/) as well as the relatively new [jspm](http://jspm.io/) – which is like npm for “browser package management”. In fact Rob and his Aurelia team are working very closely with the jspm team in order to add in functionality that will improve how Aurelia is bundled and packaged – but I digress.

To utilize npm and jspm, you need to specify the dependencies that you have on any npm/jspm packages in a packages.json file. Then you can run “npm install” and “jspm install” and the package managers spring into action pulling down all your dependencies. This works great while you’re developing – but can be a bit strange when you’re deploying with [WebDeploy](http://www.hanselman.com/blog/WebDeploymentMadeAwesomeIfYoureUsingXCopyYoureDoingItWrong.aspx) (and you should be!)

WebDeploy (out of the box) only packages files that are included in your project. This is what you want for any of your source (or content) files. But you really don’t want to include dependencies in your project (or in source control for that matter) since the package managers are going to refresh the dependencies during the build anyway. That’s the whole point of using Package Managers in the first place! The problem is that when you package your website, none of the dependencies will be included in the package (since they’re not included in the VS project).

There are a couple solutions to this problem:

1. You could execute the package manager install commands after you’ve deployed your site via WebDeploy. However, if you’re deploying to [WAWS](http://azure.microsoft.com/en-us/services/websites/) (or don’t have access to running scripts on the server where your site is hosted) you won’t be able to – and you are going to end up with missing dependencies.
2. You could include the packages folder in your project. The problem with this is that if you upgrade a package, you’ll end up having to exclude the old package (and its dependencies) and include the new package (and any of its dependencies). You lose the value of using the Package Manager in the first place.
3. Customize WebDeploy to include the packages folder when creating the deployment package. Now we’re talking!

## Including Package Folders in WebDeploy

Of course as I considered this problem I was not happy with either running the Package Manager commands on my hosting servers (in the case of WAWS this isn’t even possible) or including the package files in my project. I then searched out [Sayed Ibrahim Hashimi’s site](http://sedodream.com) to see what guidance he could offer (he’s a build guru!). I found an old post that explained how to include [“extra folders” in web deployment](http://sedodream.com/2010/03/10/WebDeploymentToolIncludingOtherFiles.aspx) – however, that didn’t quite work for me. I had to apply the slightly more up-to-date property group specified in [this post](http://www.asp.net/mvc/overview/deployment/visual-studio-web-deployment/deploying-extra-files). Sayed had a property group for \<CopyAllFilesToSingleFolderFor<u>Package</u>DependsOn\> but you need the same property group for \<CopyAllFilesToSingleFolderFor<u>Msdeploy</u>DependsOn\>.

My final customized target to include the jspm package folder in WebDeploy actions is as follows (you can add this to the very bottom of your web project file, just before the closing \</Project\> tag):

    &lt;!-- Include the jspm_packages folder when packaging in webdeploy since they are not included in the project --&gt;
    &lt;PropertyGroup&gt;
      &lt;CopyAllFilesToSingleFolderForPackageDependsOn&gt;
        CustomCollectFiles;
        $(CopyAllFilesToSingleFolderForPackageDependsOn);
      &lt;/CopyAllFilesToSingleFolderForPackageDependsOn&gt;
    
      &lt;CopyAllFilesToSingleFolderForMsdeployDependsOn&gt;
        CustomCollectFiles;
        $(CopyAllFilesToSingleFolderForPackageDependsOn);
      &lt;/CopyAllFilesToSingleFolderForMsdeployDependsOn&gt;
    &lt;/PropertyGroup&gt;
    
    &lt;Target Name="CustomCollectFiles"&gt;
      &lt;ItemGroup&gt;
        &lt;_CustomFiles Include=".\jspm_packages\**\*"&gt;
          &lt;DestinationRelativePath&gt;%(RecursiveDir)%(Filename)%(Extension)&lt;/DestinationRelativePath&gt;
        &lt;/_CustomFiles&gt;
        &lt;FilesForPackagingFromProject Include="%(_CustomFiles.Identity)"&gt;
          &lt;DestinationRelativePath&gt;jspm_packages\%(RecursiveDir)%(Filename)%(Extension)&lt;/DestinationRelativePath&gt;
        &lt;/FilesForPackagingFromProject&gt;
      &lt;/ItemGroup&gt;
    &lt;/Target&gt;

Now when I package my site, I get all the jspm packages included.

## TeamBuild with Gulp, NPM, JSPM and WebDeploy

The next challenge is getting this all to work on a TeamBuild. Let’s quickly look at what you need to do manually to get a project like this to compile:

1. Pull the sources from source control
2. Run “npm install” to install the node pacakges
3. Run “jspm install –y” to install the jspm packages
4. (Optionally) Run gulp – in our case this is required since we’re using TypeScript. We’ve got gulp set up to transpile our TypeScript source into js, do minification etc.
5. Build in VS – for our WebAPI backend
6. Publish using WebDeploy (could just be targeting a deployment package rather than pushing to a server)

Fortunately, once you’ve installed npm and jspm and gulp globally (using –g) you can create a simple PowerShell script to do steps 2 – 4. The out of the box build template does the rest for you. Here’s my Gulp.ps1 script, which I specify in the “Pre-build script path” property of my TeamBuild Process:

    param(
        [string]$sourcesDirectory = $env:TF_BUILD_SOURCESDIRECTORY
    )
    
    $webDirectory = $sourcesDirectory + "\src\MyWebProject"
    Push-Location
    
    # Set location to MyWebProject folder
    Set-Location $webDirectory
    
    # refresh the packages required by gulp (listed in the package.json file)
    $res = npm install 2&gt;&amp;1
    $errs = ($res | ? { $_.gettype().Name -eq "ErrorRecord" -and $_.Exception.Message.ToLower().Contains("err") })
    if ($errs.Count -gt 0) {
        $errs | % { Write-Error $_ }
        exit 1
    } else {
        Write-Host "Successfully ran 'npm install'"
    }
    
    # refresh the packages required by jspm (listed in the jspm section of package.json file)
    $res = jspm install -y 2&gt;&amp;1
    $errs = ($res | ? { $_.gettype().Name -eq "ErrorRecord" -and $_.Exception.Message.ToLower().Contains("err") })
    if ($errs.Count -gt 0) {
        $errs | % { Write-Error $_ }
        exit 1
    } else {
        Write-Host "Successfully ran 'jspm install -y'"
    }
    
    # explicitly set the configuration and invoke gulp
    $env:NODE_ENV = 'Release'
    node_modules\.bin\gulp.cmd build
    
    Pop-Location

One last challenge – one of the folders (a lodash folder) ends up having a path \> 260 characters. TeamBuild can’t remove this folder before doing a pull of the sources, so I had to modify the build template in order to execute a “CleanNodeDirs” command (I implemented this as an optional “pre-pull” script). However, this is a chicken-and-egg problem – if the pull fails because of old folders, then you can’t get the script to execute to clean the folders before the pull… So the logic I wrap the “pre-pull” invocation in an If activity that first checks if the “pre-pull” script exists. If it does, execute it, otherwise carry on.

The logic for this is as follows:

1. On a clean build (say a first build) the pre-pull script does not exist
2. When the build checks for the pre-pull script, it’s not there – the build continues
3. The build executes jspm, and the offending lodash folder is created
4. The next build initializes, and detects that the pre-pull script exists
5. The pre-pull script removes the offending folders
6. The pull and the remainder of the build can now continue

Unfortunately straight PowerShell couldn’t delete the folder (since the path is \> 260 chars). I resorted to invoking cmd. I repeat it twice since the first time it complains that the folder isn’t empty – running the 2nd time completes the delete. Here’s the script:

    Param(
      [string]$srcDir = $env:TF_BUILD_SOURCESDIRECTORY
    )
    
    # forcefully remove left over node module folders
    # necessary because the folder depth means paths end up being &gt; 260 chars
    # run it twice since it sometimes complains about the dir not being empty
    # supress errors
    $x = cmd /c "rd $srcDir\src\MyWebProject\node_modules /s /q" 2&gt;&amp;1
    $x = cmd /c "rd $srcDir\src\MyWebProject\node_modules /s /q" 2&gt;&amp;1

## Conclusion

Getting NPM, JSPM, Gulp, WebDeploy and TeamBuild to play nicely is not a trivial exercise. Perhaps vNext builds will make this all easier – I’ve yet to play with it. For now, we’re happy with our current process.

Any build/deploy automation can be tricky to set up initially – especially if you’ve got as many moving parts as we have in our solution. However, the effort pays off, since you’ll be executing the build/deploy cycle many hundreds of times over the lifetime of an agile project – each time you can deploy from a single button-press is a win!

Happy packaging!

