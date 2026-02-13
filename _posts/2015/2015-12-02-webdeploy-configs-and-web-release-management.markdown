---
layout: post
title: WebDeploy, Configs and Web Release Management
date: '2015-12-02 19:18:38'
tags:
- releasemanagement
- build
---

It’s finally here – the new [web-based Release Management](https://www.visualstudio.com/en-us/get-started/release/release-management-vs) (WebRM). At least, it’s here in preview on VSTS (formerly VSO) and should hopefully come to TFS 2015 in update 2.

I’ve blogged frequently about Release Management, the “old” WPF tool that Microsoft purchased from InCycle (it used to be called InRelease). The tool was good in some ways, and horrible in others – but it always felt like a bit of a stop-gap while Microsoft implemented something truly great – which is what WebRM is!

One of the most common deployment scenarios is deploying web apps – to IIS or to Azure. I blogged about using the old tool along with WebDeploy [here](/webdeploy-and-release-management--the-proper-way). This post is a follow-on – how to use WebDeploy and WebRM correctly.

First I want to outline a problem with the out-of-the-box Tasks for deploying web apps. Then I’ll talk about how to tokenize the build package ready for multi-environment deployments, and finally I’ll show you how to create a Release Definition.

## Azure Web App Deployment Task Limitations

If you create a new Release Definition, there is an “Azure Web App Deployment” Task. Why not just use that to deploy web apps? There are a couple of issues with this Task:

1. You can’t use it to deploy to IIS
2. You can’t manage different configurations for different environments (with the exception of connection strings)

The Task is great in that it uses a predefined Azure Service Endpoint, which abstracts credentials away from the deployment. However, the underlying implementation invokes an Azure PowerShell cmdlet [Publish-AzureWebsiteProject](https://msdn.microsoft.com/en-us/library/azure/dn722468.aspx). This cmdlet works – as long as you don’t intend to change any configuration except the connection strings. Have different app settings in different environments? You’re hosed. Here’s the Task UI in VSTS:

<!--kg-card-begin: html-->[![image](/assets/images/files/54e7111d-5892-44e8-b0f8-f2eb7f277d3b.png "image")](/assets/images/files/2a26c2ff-f9a0-4dc4-9619-4111a7e8e9ab.png)<!--kg-card-end: html-->

The good:

- You select the Azure subscription from the drop-down – no messing with passwords
- You can enter a deployment slot

The bad:

- You have to select the zip file for the packaged site – no place for handling configs
- Additional arguments – almost impossible to figure out what to put here. You can use this to set connection strings if you’re brave enough to figure it out

The ugly:

- Web App Name is a combo-box, but it’s never populated, so you have to type the name yourself (why is it a combo-box then?)

In short, this demo’s nicely, but you’re not really going to use it for any serious deployments – unless you’ve set the app settings on the slots in the Azure Portal itself. Perhaps this will work for you – but if you change a setting value (or add a new setting) you’re going to have to manually update the slot using the Portal. Not a great automation story.

## Config Management

So besides not being able to use the Task for IIS deployments, your biggest challenge is config management. Which is ironic, since building a WebDeploy package actually handles the config well – it places config into a SetParameters.xml file. Unfortunately the Task (because it is calling Publish-AzureWebsiteProject under the hood) only looks for the zip file – it ignores the SetParameters file.

So I got to thinking – and I stole an idea from [Octopus Deploy](https://octopus.com/): what if the deployment would just automagically replace any config setting value with any correspondingly named variable defined in the Release Definition for the target Environment? That would mean you didn’t have to edit long lists of arguments at all. Want a new value? Just add it to the Environment variables and the deployment takes care of it for you.

## The Solution

The solution turned out to be fairly simple:

For the VS Solution:

1. Add a parameters.xml file to your Website project for any non-connecting string settings you want to manage, using tokens for values
2. Create a publish profile that inserts tokens for the website name and any db connection strings

For the Build:

1. Configure a Team Build to produce the WebDeploy package (and cmd and SetParameters files) using the publish profile
2. Configure the Build to upload the zip and supporting files as the output

For the Release:

1. Write a script to do the parameter value substitution (replacing tokens with actual values defined in the target Environment) into the SetParameters file
2. Invoke the cmd to deploy the Website

Of course, the “parameter substituting script” has to be checked into the source repo and also included as a build output in order for you to use it in the Release.

### Creating a Tokenized WebDeploy Package in a Team Build

Good releases start with good packages. Since the same package is going to be deployed to multiple environments, you cannot “hardcode” any config settings into the package. So you have to create the package in such a way that it has tokens for any config values that the Release pipeline will replace with Environment specific values at deployment time. In my previous [WebDeploy and Release Management post](/webdeploy-and-release-management--the-proper-way), I explain how to add the parameters.xml file and how to create a publish profile to do exactly that. That technique stays exactly the same as far as the VS solution goes.

Here’s my sample parameters.xml file for this post:

    &lt;!--?xml version="1.0" encoding="utf-8" ?--&gt;
    &lt;parameters&gt;
      &lt;parameter name="CoolKey" description="The CoolKey setting" defaultvalue=" __CoolKey__" tags=""&gt;
        &lt;parameterentry kind="XmlFile" scope="\\web.config$" match="/configuration/appSettings/add[@key='CoolKey']/@value"&gt;
        &lt;/parameterentry&gt;
      &lt;/parameter&gt;
    &lt;/parameters&gt;

Note how I’m sticking with the double-underscore pre- and post-fix as the token, so the value (token) for CoolKey is \_\_CoolKey\_\_.

Once you’ve got a parameters.xml file and a publish profile committed into your source repo (Git or TFVC – either one works fine), you’re almost ready to create a Team Build (vNext Build). You will need the script that “hydrates” the parameters from the Environment variables. I’ll cover the contents of that script shortly – let’s assume for now that you have a script called “Replace-SetParameters.ps1” checked into your source repo along with your website. Here’s the structure I use:

<!--kg-card-begin: html-->[![image](/assets/images/files/0ca3df4c-51ba-486a-ae98-772b7034b779.png "image")](/assets/images/files/47811e0e-47d3-4131-a14c-4da0c5fd828f.png)<!--kg-card-end: html-->

Create a new Build Definition – select Visual Studio Build as the template to start from. You can then configure whatever you like in the build, but you have to do 3 things:

1. Configure the MSBuild arguments as follows in the “Visual Studio Build” Task:
2. /p:DeployOnBuild=true /p:PublishProfile=Release /p:PackageLocation="$(build.StagingDirectory)"
3. The name of the PublishProfile is the same name as the pubxml file in your solution
4. The package location is set to the build staging directory
<!--kg-card-begin: html-->[![image](/assets/images/files/4d9e81c0-4cd6-4685-b27d-080a7fa2b106.png "image")](/assets/images/files/2a5f8f9c-c7cf-4264-b22b-351cb07046bb.png)<!--kg-card-end: html-->
1. Configure the “Copy and Publish Build Artifacts” Task to copy the staging directory to a server drop:
<!--kg-card-begin: html-->[![image](/assets/images/files/6f8812b9-142f-434b-a6f9-4b060630517d.png "image")](/assets/images/files/a249909e-a879-4e40-9ef2-a2a3cff04ad2.png)<!--kg-card-end: html-->
1. Add a new “Publish Build Artifact” Task to copy the “Replace-SetParameters.ps1” script to a server drop called “scripts”:
<!--kg-card-begin: html-->[![image](/assets/images/files/66fb8d87-a768-4adf-8ff8-6ab42fcfcf50.png "image")](/assets/images/files/664fa560-0f9f-4034-b959-b97b5b5d88e3.png)<!--kg-card-end: html-->

I like to version my assemblies so that my binary versions match my build number. I use a [custom build Task](https://github.com/colindembovsky/cols-agent-tasks/tree/master/Tasks/VersionAssemblies) to do just that. I also run unit tests as part of the build. Here’s my entire build definition:

<!--kg-card-begin: html-->[![image](/assets/images/files/4c86556e-4b23-4852-9aa3-9c0bf78486a8.png "image")](/assets/images/files/ddd96c1f-4c66-4cfe-b666-202738ffe9b9.png)<!--kg-card-end: html-->

Once the build has completed, the Artifacts look like this:

<!--kg-card-begin: html-->[![image](/assets/images/files/f90dcc60-02cc-429e-9016-9b401a7eb6e1.png "image")](/assets/images/files/499b4107-1af6-4272-bbc1-a8d5fb0c4494.png)<!--kg-card-end: html--><!--kg-card-begin: html-->[![image](/assets/images/files/1ec03479-9722-4704-a3bd-a028d1663996.png "image")](/assets/images/files/374f429e-bcb4-49e4-9a94-4760f4703ae3.png)<!--kg-card-end: html--><!--kg-card-begin: html-->[![image](/assets/images/files/55f87666-1e7e-4228-91e7-260b515a0597.png "image")](/assets/images/files/cfddee21-bdd1-4a0e-bbde-6bd7e61560b9.png)<!--kg-card-end: html-->

Here’s what the SetParameters file looks like if you open it up:

    &lt;?xml version="1.0" encoding="utf-8"?&gt;
    &lt;parameters&gt;
      &lt;setParameter name="IIS Web Application Name" value=" __SiteName__" /&gt;
      &lt;setParameter name="CoolKey" value=" __CoolKey__" /&gt;
      &lt;setParameter name="EntityDB-Web.config Connection String" value=" __EntityDB__" /&gt;
    &lt;/parameters&gt;

The tokens for SiteName and EntityDB both come from my publish profile – the token for CoolKey comes from my parameters.xml file.

Now we have a package that’s ready for Release!

### Filling in Token Values

You can see how the SetParameters file contains tokens. We will eventually define values for each token for each Environment in the Release Definition. Let’s assume that’s been done already – then how does the release pipeline perform the substitution? Enter PowerShell!

When you execute PowerShell in a Release, any Environment variables you define in the Release Definition are created as environment variables that the script can access. So I wrote a simple script to read in the SetParameters file, use Regex to find any tokens and replace the tokens with the environment variable value. Of course I then overwrite the file. Here’s the script:

    param(
        [string]$setParamsFilePath
    )
    Write-Verbose -Verbose "Entering script Replace-SetParameters.ps1"
    Write-Verbose -Verbose ("Path to SetParametersFile: {0}" -f $setParamsFilePath)
    
    # get the environment variables
    $vars = gci -path env:*
    
    # read in the setParameters file
    $contents = gc -Path $setParamsFilePath
    
    # perform a regex replacement
    $newContents = "";
    $contents | % {
        $line = $_
        if ($_ -match "__(\w+)__") {
            $setting = gci -path env:* | ? { $_.Name -eq $Matches[1] }
            if ($setting) {
                Write-Verbose -Verbose ("Replacing key {0} with value from environment" -f $setting.Name)
                $line = $_ -replace "__(\w+)__", $setting.Value
            }
        }
        $newContents += $line + [Environment]::NewLine
    }
    
    Write-Verbose -Verbose "Overwriting SetParameters file with new values"
    sc $setParamsFilePath -Value $newContents
    
    Write-Verbose -Verbose "Exiting script Replace-SetParameters.ps1"

Notes:

- Line 2: The only parameter required is the path to the SetParameters file
- Line 8: Read in all the environment variables – these are populated according to the Release Definition
- Line 11: Read in the SetParameters file
- Line 15: Loop through each line in the file
- Line 17: If the line contains a token, then:
- Line 18-22: Find the corresponding environment variable, and if there is one, replace the token with the value
- Line 27: Overwrite the SetParameters file

Caveats: note, this can be a little bit dangerous since the environment variables that are in scope include more than just the ones you define in the Release Definition. For example, the environment includes a “UserName” variable, which is set to the build agent user name. So if you need to define a username variable, make sure you name it “WebsiteUserName” or something else that’s going to be unique.

## Creating the Release Definition

We now have all the pieces in place to create a Release Definition. Each Environment is going to execute (at least) 2 tasks:

- PowerShell – to call the Replace-SetParameters.ps1 script
- Batch Script – to invoke the cmd file to publish the website

The PowerShell task is always going to be exactly the same – however, the Batch Script arguments are going to change slightly depending on if you’re deploying to IIS or to Azure.

I wanted to make sure this technique worked for IIS as well as for Azure (both deployment slots and “real” sites). So in this example, I’m deploying to 3 environments: Dev, Staging and Production. I’m using IIS for dev, to a staging deployment slot in Azure for Staging and the “real” Azure site for Production.

Here are the steps to configure the Release Definition:

1. Go to the Release hub in VSTS and create a new Release Definition. Select “Empty” to start with an empty template.
2. Enter a name for the Release Definition and change “Default Environment” to Dev
<!--kg-card-begin: html-->[![image](/assets/images/files/218622b6-48aa-453c-92d3-eb5d9884aacf.png "image")](/assets/images/files/e75f4709-35f4-44d0-aef2-b2dfbbe64479.png)<!--kg-card-end: html-->
1. Click “Link to a Build Definition” and select the build you created earlier:
<!--kg-card-begin: html-->[![image](/assets/images/files/509668ce-0c8d-4c0d-807c-e509c7f02c8b.png "image")](/assets/images/files/7a9168c1-a851-4693-a267-6c6d420c42d8.png)<!--kg-card-end: html-->
1. Click “+ Add Tasks” and add a PowerShell Task:
2. For the “Script filename”, browse to the location of the Replace-SetParameters.ps1 file:
<!--kg-card-begin: html-->[![image](/assets/images/files/45f0a4f9-0cd8-40af-bc7b-b26dd6fdb872.png "image")](/assets/images/files/61cc8620-e0f0-49e4-9e52-95bc65246bc6.png)<!--kg-card-end: html-->
1. For the “Arguments”, enter the following:
2. -setParamsFilePath $(System.DefaultWorkingDirectory)\CoolWebApp\drop\CoolWebApp.SetParameters.xml
3. Of course you’ll have to fix the path to set it to the correct SetParameters file – $(System.DefaultWorkingDirectory) is the root of the Release downloads. Then there is a folder with the name of the Build (e.g. CoolWebApp), then the artifact name (e.g. drop), then the path within the artifact source.
4. Click “+ Add Tasks” and add a Batch Script Task:
5. For the “Script filename”, browse to the location of the WebDeploy cmd file:
<!--kg-card-begin: html-->[![image](/assets/images/files/5b4b9268-fc4e-408a-99bb-5d0899151fd4.png "image")](/assets/images/files/58caff03-e790-4eec-96e1-8f9a4fa15a70.png)<!--kg-card-end: html-->
1. Enter the correct arguments (discussed below).
2. Configure variables for the Dev environment by clicking the ellipses button on the Environment tile and selecting “Configure variables”
3. Here you add any variable values you require for your web app – these are the values that you tokenized in the build:
<!--kg-card-begin: html-->[![image](/assets/images/files/67406726-55b5-4513-803d-7f9c372cfbf7.png "image")](/assets/images/files/c53f51fe-d3eb-4e92-9fd8-81875695b77c.png)<!--kg-card-end: html-->
1. Azure sites require a username and password – I’ll cover those shortly.

The Definition should now look something like this:

<!--kg-card-begin: html-->[![image](/assets/images/files/854cc6b2-2d55-492f-bd3a-1ab7afd8de10.png "image")](/assets/images/files/3515cdc0-e574-4893-87d0-5313b84ef2ab.png)<!--kg-card-end: html-->
### Cmd Arguments and Variables

For IIS, you don’t need username and password for the deployments. This means you’ll need to configure the build agent to run as an identity that has permissions to invoke WebDeploy. The SiteName variable is going to be the name of the website in IIS plus the name of your virtual application – something like “Default Web Site/cool-webapp”. Also, you’ll need to configure the Agent on the Dev environment to be an on-premise agent (so select an on-premise queue) since the hosted agent won’t be able to deploy to your internal IIS servers.

For Azure, you’ll need the website username and password (which you can get by downloading the Publish profile for the site from the Azure Portal). They’ll need to be added as variables in the environment, along with another variable called “WebDeploySiteName” (which is required only if you’re using deployment slots). The SiteName is going to be the name of the site in Azure. Of course you’re going to “lock” the password field to make it a secret. You can use the Hosted agent for Environments that deploy to Azure.

Here are the 2 batch commands – the first is for local deployment to IIS, the 2nd for deployment to Azure:

- /Y /M:http://$(WebDeploySiteName)/MsDeployAgentService
- /Y /M:https://$(WebDeploySiteName).scm.azurewebsites.net:443/msdeploy.axd /u:$(AzureUserName) /p:$(AzurePassword) /a:Basic

For IIS deployments, you can set WebDeploySiteName to be the name or IP of the target on-premises server. Note that you’ll have to have WebDeploy remote agent running on the machine, with the appropriate permissions for the build agent identity to perform the deployment.

For Azure, the WebDeploySiteName is of the form “siteName[-slot]”. So if you have a site called “MyWebApp”, and you just want to deploy to the site, then WebDeploySiteName will be “MyWebApp”. If you want to deploy to a slot (e.g. Staging), then WebDeploySiteName must be set to “MyWebApp-staging”. You’ll also need to set the SiteName to the name of the site in Azure (“MyWebApp” for the site, “MyWebApp\_\_slot” for a slot – e.g. “MyWebApp\_\_staging”). Finally, you’ll need “AzureUserName” and “AzurePassword” to be set (according to the publish settings for the site).

#### Cloning Staging and Production Environments

Once you’re happy with the Dev Environment, clone it to Staging and update the commands and variables. Then repeat for Production. You’ll now have 3 Environments in the Definition:

<!--kg-card-begin: html-->[![image](/assets/images/files/ed956818-8992-49a9-b73e-d4ca69dd7dd0.png "image")](/assets/images/files/ffd5137d-6ee4-4330-a9cd-1fc1401bffdc.png)<!--kg-card-end: html-->

Also, if you click on “Configuration”, you can see all the Environment variables by clicking “Release variables” and selecting “Environment Variables”:

<!--kg-card-begin: html-->[![image](/assets/images/files/886e0140-1a69-41cb-9004-8c3df029bc7e.png "image")](/assets/images/files/ceef56c7-7946-4390-94b2-a3fd237b6d7f.png)<!--kg-card-end: html-->

That will open a grid so you can see all the variables side-by-side:

<!--kg-card-begin: html-->[![image](/assets/images/files/62df6094-71eb-4416-b260-e4afe6b260f9.png "image")](/assets/images/files/86ece78f-6886-40a8-951a-934cac54fc2d.png)<!--kg-card-end: html-->

Now you can ensure that you’ve set each Environment’s variables correctly. Remember to set approvals on each environment as appropriate!

### 2 More Tips

If you want to trigger the Release every time the linked Build produces a new package, then click on Triggers and enable “Continuous Deployment”.

You can get the Release number to reflect the Build package version. Click on General and change the Release Name format to:

<!--kg-card-begin: html--><font face="Courier New">$(Build.BuildNumber)-$(rev:r)</font><!--kg-card-end: html-->

Now when you release 1.0.0.8, say, your release will be “1.0.0.8-1”. If you trigger a new release with the same package, it will be numbered “1.0.0.8-2” and so on.

## Conclusion

WebRM is a fantastic evolution of Release Management. It’s much easier to configure Release Definitions, to track logs to see what’s going on and to configure deployment Tasks – thanks to the fact that the Release agent is the same as the Build agent. As far as WebDeploy goes, I like this technique of managing configuration – I may write a custom Build Task that bundles the PowerShell and Batch Script into a single task – that will require less argument “fudging” and bundle the PowerShell script so you don’t have to have it in your source repo. However, the process is not too difficult to master even without a custom Task, and that’s pleasing indeed!

Happy releasing!

