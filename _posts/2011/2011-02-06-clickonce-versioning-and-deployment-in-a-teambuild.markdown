---
layout: post
title: ClickOnce Versioning and Deployment in a TeamBuild
date: '2011-02-06 23:49:00'
tags:
- build
---

TeamBuild in TFS 2010 is an incredibly powerful engine – it’s based on Windows Workflow 4 (WF4). Once you get over the initial learning curve of WF4, you can get your team builds to do some impressive stuff.

One common scenario is supporting a ClickOnce deployment from a build. Essentially, you need to add an MSBuild activity into your build, tweaking the command line parameters that are passed to MSBuild. That’s not too difficult in and of itself – but I wanted to go further and tie the ClickOnce version into the build version.

To do that, I used a custom build activity from [Jim Lamb’s post on versioning assemblies](http://blogs.msdn.com/b/jimlamb/archive/2009/11/18/how-to-create-a-custom-workflow-activity-for-tfs-build-2010.aspx). However, I had to tweak the “UpdateVersionInfo” activity to accept 2 regex patterns instead of Jim’s 1. Jim uses the regex pattern to extract version info from the build number (so if your build is ‘MyBuild\_4.0.0.2’ he extracts 4.0.0.2). He uses the same pattern to find/replace in the AssemblyInfo.\* files (or whatever files you specify). Usually this is fine.

## Creating the Custom Assembly

However, the assembly version in the project file for a ClickOnce app can have a different format – if you select “increment revision whenever I deploy” in the Publish properties of a ClickOnce project, the version places a \* in the revision number – making the version 1.0.0.\* (or if you look in the project file, 1.0.0.%2a). So I needed to separate the “source” regex pattern (the pattern used to extract the version from the build number) and the “target” regex – the pattern used for the search/replace in the target files.

I opened the UpdateVersionInfo.xaml and added a new argument (called “SourceRegeEx”). I then changed the “Extract Version Info” assignment activity to use SourceRegex instead of Regex.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/_d41Ixos7YsM/TU6mp0IpFDI/AAAAAAAAAM8/QhbvuEBwMj8/image_thumb4.png?imgmax=800 "image")](http://lh4.ggpht.com/_d41Ixos7YsM/TU6mm9ayV8I/AAAAAAAAAM4/5gJJEueEjYY/s1600-h/image8.png)<!--kg-card-end: html-->

In the custom build workflow (make sure you follow Jim’s method of copying and then branching your custom workflow) I used two UpdateVersionInfo activities – one to update the version info in the AssemblyInfo.\* files and one to update the ClickOnce vcproj file (this is where the ClickOnce version resides).

## Customizing the Default Build Template

I stared from the DefaultTemplate.xaml added a number of arguments (I also updated the metadata to make the arguments prettier when you create builds):

- VersioningFileSpec – the file search pattern to update assembly versions (AssemblyInfo.\* by default)
- VersioningRegularExpression – the regex pattern to use to update assembly versions (“\d+\.\d+\.\d+\.\d+” by default)
- ClickOnceDeployIfTestsFail – true to deploy even if tests fail, false otherwise
- ClickOncePublishDir – the UNC share to publish to
- ClickOncePublishURL – the ClickOnce URL (can be the same as the PublishDir)
- ClickOnceCertThumbprint – the thumbprint of the certificate for signing manifests (note: this certificate needs to be installed on any build agent machines – you can copy the pfx file (if you made a temporary certificate in VS) to the build machine and import it into the store by double clicking it – then use mmc to view the certificate and get its SHA1 for this field when you create a build)
- ClickOnceSolutionPath – the source control path of the solution that contains the ClickOnce project
- ClickOnceVersionFileSpec – the file search pattern to update ClickOnce version – usually the name of the csproj file that you want to publish – e.g. ClickOnceProject.csproj)
- ClickOnceVersionRegEx – the regex target pattern for doing ClickOnce version replacements (usually either “\d+.\d+\.\d+\.%2a” or “\d+\.\d+\.\d+\.\d+”)

I then added two UpdateVersionInfo activities – one right after the “GetWorkspace Activity” in the Process-\>Overall Build Process-\>Run On Agent-\>Initialize Workspace sequence. I used the VersioningFileSpec and RegularExpression arguments and hardcoded the source regex to \d+\.\d+\.\d+\.\d+”.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/_d41Ixos7YsM/TU6m2b4t0uI/AAAAAAAAANE/Q1pREKrCWh4/image_thumb7.png?imgmax=800 "image")](http://lh5.ggpht.com/_d41Ixos7YsM/TU6mx8JP44I/AAAAAAAAANA/lpmZYeUn1OQ/s1600-h/image13.png)<!--kg-card-end: html-->

The next step was the ClickOnce versioning customizations. I located the step “If TestStatus = Unknown” in the “Try Compile and Test” sequence. Right after that activity I added a sequence activity called “ClickOnce Deployment”.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/_d41Ixos7YsM/TU6m6M0trZI/AAAAAAAAANM/rPBq1LW9jno/image_thumb15.png?imgmax=800 "image")](http://lh4.ggpht.com/_d41Ixos7YsM/TU6m4Poxz3I/AAAAAAAAANI/kAxjxoAvCAA/s1600-h/image20.png)<!--kg-card-end: html-->

Here is the sequence and explanation:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/_d41Ixos7YsM/TU6m9TkNcDI/AAAAAAAAANU/dR9OVFE58a0/image_thumb23.png?imgmax=800 "image")](http://lh4.ggpht.com/_d41Ixos7YsM/TU6m7vM3FoI/AAAAAAAAANQ/pgFOvVG8KTg/s1600-h/image25.png)<!--kg-card-end: html-->
- Add an “If” activity
- Condition: ClickOnceDeployIfTestsFail Or BuildDetail.TestStatus = BuildPhaseStatus.Succeeded
- This will ensure that the “deploy even if failed” condition is obeyed
- Add a “WriteBuildMessage” to the Else side of the If Activity to just log that tests failed and the deployment will be skipped
- Add a sequence to the “Then” side of the If
- Add an UpdateVersionInfo Activity (this will apply the build version to the ClickOnce deployment) and set the following:
- SourcesDirectory = SourcesDirectory
- FileSpec = ClickOnceVersionFileSpec
- SourceRegEx = "\d+\.\d+\.\d+\.\d+"
- RegularExpression = ClickOnceVersionRegEx
- Add a ConvertWorkspaceItem (to convert the source path of the click once solution to a local path) and set:
- Direction = ServerToLocal
- Input = ClickOnceSolutionPath
- Result = localSolution (you’ll need to add this as a string variable on the sequence)
- Workspace = Workspace
- Add an MSBuild Activity to publish
- The only argument worthy of note here is the CommandLineArguments which should be set to:String.Format("/p:SkipInvalidConfigurations=true {0} /Target:Publish /property:PublishDir=""{1}"" /property:PublishUrl=""{2}"" /property:InstallUrl=""{2}"" /property:SignManifests=true /property:ManifestCertificateThumbprint=""{3}""", MSBuildArguments, ClickOncePublishDir, ClickOncePublishURL, ClickOnceCertThumbprint)
- Add a WriteBuildMessage to say that the deployment was executed

Now you need to compile the custom activity project and put the dll into source control for your controller to find (again refer to Jim’s blog post).

Finally, you can set up a build:

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/_d41Ixos7YsM/TU6nBJM5x_I/AAAAAAAAANc/MTLCadbLM_g/image_thumb33.png?imgmax=800 "image")](http://lh6.ggpht.com/_d41Ixos7YsM/TU6m-hShNKI/AAAAAAAAANY/naO1_n59miU/s1600-h/image36.png)<!--kg-card-end: html-->

NOTE: If you have different configurations (for Production and Staging for example) use [Vishal Joshi’s blog](http://vishaljoshi.blogspot.com/2010/05/applying-xdt-magic-to-appconfig.html) to enable config transforms on your app.config file – then target the config you want in the build.

Download the project (including the DefaultClickOnce.xaml template – see the test project’s Templates folder) from my [skydrive](http://cid-64a24e0938d6d062.office.live.com/self.aspx/Colin%5E4s%20ALM%20Corner/ClickOnceBuild/ClickOnceBuildActivityPack.zip).

