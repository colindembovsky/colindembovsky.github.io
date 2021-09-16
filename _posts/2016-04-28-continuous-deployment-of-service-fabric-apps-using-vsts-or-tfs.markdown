---
layout: post
title: Continuous Deployment of Service Fabric Apps using VSTS (or TFS)
date: '2016-04-28 19:41:34'
tags:
- devops
- cloud
- build
---

Azure’s [Service Fabric](https://azure.microsoft.com/en-us/documentation/articles/service-fabric-overview/) is breathtaking – the platform allows you to create truly “born in the cloud” apps that can _really_ scale. The platform takes care of the plumbing for you so that you can concentrate on business value in your apps. If you’re looking to create cloud apps, then make sure you take some time to investigate Service Fabric.

## Publishing Service Fabric Apps

Unfortunately, most of the samples (like this [getting started one](https://github.com/azure-samples/service-fabric-dotnet-getting-started) or this more [real-world one](https://github.com/Azure-Samples/service-fabric-dotnet-web-reference-app)) don’t offer any guidance around continuous deployment. They just wave hands and say, “Publish from Visual Studio” or “Publish using PowerShell”. Which is all well and good – but how do you actually do proper DevOps with ServiceFabric Apps?

Publishing apps to Service Fabric requires that you _package_ the app and then _publish_ it. Fortunately VSTS allows you to fairly easily package the app in an automated build and then publish the app in a release.

There are two primary challenges to doing this:

1. Versioning. Versioning is critical to Service Fabric apps, so your automated build is going to have to know how to version the app (and its constituent services) correctly
2. Publishing – new vs upgrade. The out-of-the-box publish script (that you get when you do a File-\>New Service Fabric App project) needs to be invoked differently for new apps as opposed to upgrading existing apps. In the pipeline, you want to be able to publish the same way – whether or not the application already exists. Fortunately a couple modifications to the publish script do the trick.

Finally, the cluster should be created or updated on the fly during the release – that’s what the ARM templates do.

To demonstrate a Service Fabric build/release pipeline, I’m going to use a “fork” of the original [VisualObjects](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/tree/master/Actors/VisualObjects) sample from the [getting started repo](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started) (it’s not a complete fork since I just wanted this one solution from the repo). I’ve added an ARM template project to demonstrate how to create the cluster using ARM during the deployment and then I’ve added two publishing profiles – one for Test and one for Prod. The ARM templates and profiles for both Test and Prod are exactly the same in the repo – in real life you’ll have a beefier cluster in Prod (with different application parameters) than you will in test, so the ARM templates and profiles are going to look different. Having two templates and profiles gives you the idea of how to separate environments in the Release, which is all I want to demonstrate.

This entire flow works on TFS as well as VSTS, so I’m just going to show you how to do this using VSTS. I’ll call out differences for TFS when necessary.

### Getting the Code

The easiest way is to just fork [this repo](https://github.com/colindembovsky/ServiceFabricBuildRelease) on Github. You can of course clone the repo, then push it to a VSTS project if you prefer. For this post I’m going to use code that I’ve imported into a VSTS repo. If you’re on TFS, then it’s probably easiest to clone the repo and push it to your TFS server.

## Setting up the Build

Unfortunately the Service Fabric SDK isn’t installed on the hosted agent image in VSTS, so you’ll have to use a private agent. Make sure the Service Fabric SDK is installed on the build machine. Use this [help doc](https://azure.microsoft.com/en-us/documentation/articles/service-fabric-get-started/) to get the bits.

The next thing you’ll need is my [VersionAssemblies custom build task](https://github.com/colindembovsky/cols-agent-tasks/tree/master/Tasks/VersionAssemblies). I’ve bundled it into a [VSTS marketplace extension](https://marketplace.visualstudio.com/items?itemName=colinsalmcorner.colinsalmcorner-buildtasks). If you’re on VSTS, just click “Install” – if you’re on TFS, you’ll need to download it and upload it. You’ll only be able to do this on TFS 2015 Update 2 or later.

Now go to your VSTS account and navigate to the Code hub. Create a new Build definition using the Visual Studio template. Select the appropriate source repo and branch (I’m just going to use master) and select the queue with your private agent. Select Continuous Integration to queue the build whenever a commit is pushed to the repo:

<!--kg-card-begin: html-->[![image](/assets/images/files/4a7acc11-01a8-47da-acaf-ad96ddc03ac0.png "image")](/assets/images/files/a1859eb3-7f2a-4937-be6c-a49dba822a8d.png)<!--kg-card-end: html-->

Change the name of the build – I’ve called mine “VisualObjects”. Go to the General tab and change the build number format to be

<!--kg-card-begin: html--><font face="Courier New">1.0$(rev:.r)</font><!--kg-card-end: html-->

This will give the build number 1.0.1, then 1.0.2, 1.0.3 and so on.

Now we want to change the build so that it will match the ApplicationTypeVersion (from the application manifest) and all the Service versions within the ServiceManifests for each service within the application. So click “Add Task” and add two “VersionAssembly” tasks. Drag them to the top of the build (so that they are the first two tasks executed).

Configure the first one as follows:

<!--kg-card-begin: html-->[![image](/assets/images/files/8122a0ca-f3c9-4819-b976-fe95f71ab0ce.png "image")](/assets/images/files/4354826f-09b8-44e2-a2c5-d9d248012266.png)<!--kg-card-end: html-->

Configure the second one as follows:

<!--kg-card-begin: html-->[![image](/assets/images/files/99b52c32-9907-4ad7-ac8b-674d86250a32.png "image")](/assets/images/files/2f967a07-15ca-493c-bea0-2279e02acd5c.png)<!--kg-card-end: html-->

The first task finds the ApplicationManifest.xml file and replaces the version with the build number. The second task recursively finds all the ServiceManifest.xml files and then also replaces the version number of each service with the build number. After the build, the application and service versions will all match the build number.

The next 3 tasks should be “NuGet Installer”, “Visual Studio Build” and “Visual Studio Test”. You can leave those as is.

Add a new “Visual Studio Build” task and place it just below the test task. Configure the Solution parameter to the path of the .sfproj in the solution (src/VisualObjects/VisualObjects/VisualObjects.sfproj). Make the MSBuild Arguments parameter “/t:Package). Finally, add $(BuildConfiguration) to the Configuration parameter. This task invokes Visual Studio to package the Service Fabric app:

<!--kg-card-begin: html-->[![image](/assets/images/files/dd5d1ce8-3ff4-4c9d-96c0-72481c6c2576.png "image")](/assets/images/files/14608904-483f-496c-9218-15c366b5d685.png)<!--kg-card-end: html-->

Now you’ll need to do some copying so that we get all the files we need into the artifact staging directory, ready for publishing. Add a couple “Copy” tasks to the build and configure them as follows:

<!--kg-card-begin: html-->[![image](/assets/images/files/7dc9fa90-26f4-4c43-824a-64b16fbe41d4.png "image")](/assets/images/files/25e4f774-60ad-46a7-9258-38be03773928.png)<!--kg-card-end: html-->

This copies the Service Fabric app package to the staging directory.

<!--kg-card-begin: html-->[![image](/assets/images/files/8eb44813-6d32-420a-8cc1-2421c9b7376e.png "image")](/assets/images/files/277b9095-4897-4410-b44a-67af02bc3389.png)<!--kg-card-end: html-->

This copies the Scripts folder to the staging directory (we’ll need this in the release to publish the app).

<!--kg-card-begin: html-->[![image](/assets/images/files/e7d5f48e-3e9f-4679-a22a-fb6e617badce.png "image")](/assets/images/files/4f246bbb-567b-4338-95b1-86091360c51a.png)<!--kg-card-end: html--><!--kg-card-begin: html-->[![image](/assets/images/files/57a3cd79-ebc2-4fa8-bd8b-fc0ba3d6542e.png "image")](/assets/images/files/fe5104b1-f723-4667-b0bb-fabfe59965be.png)<!--kg-card-end: html-->

These tasks copy the Publish Profiles and ApplicationParameters files to the staging directory. Again, these are needed for the release.

You’ll notice that there isn’t a copy task for the ARM project – that’s because the ARM project automagically puts its output into the staging directory for you when building the solution.

You can remove the Source Symbols task if you want to – it’s not going to harm anything if it’s there. If you really want to keep the symbols you’ll have to specify a network share for the symbols to be copied to.

Finally, make sure that your “Publish Build Artifacts” task is configured like this:

<!--kg-card-begin: html-->[![image](/assets/images/files/17e165bf-d097-4bc0-96b9-fab4ca7e1304.png "image")](/assets/images/files/d65f9c73-a569-4b82-887f-246cb6823c2b.png)<!--kg-card-end: html-->

Of course you can also choose a network folder rather than a server drop if you want. The tasks should look like this:

<!--kg-card-begin: html-->[![image](/assets/images/files/94c65e36-bd1f-425b-906a-c03cb0f048d6.png "image")](/assets/images/files/38590fa3-9e30-455e-b727-601734367fe0.png)<!--kg-card-end: html-->

Run the build to make sure that it’s all happy. The artifacts folder should look like this:

<!--kg-card-begin: html-->[![image](/assets/images/files/f48745e5-8e7e-4a99-9fa3-4eade351e3dd.png "image")](/assets/images/files/7a0e92a5-c56d-43d4-a725-8cf01ddbf37d.png)<!--kg-card-end: html-->
## Setting up the Release

Now that the app is packaged, we’re almost ready to define the release pipeline. There’s a decision to make at this point: to ARM or not to ARM. In order to create the Azure Resource Group containing the cluster from the ARM template, VSTS will need a secure connection to the Azure subscription (follow [these instructions](https://blogs.msdn.microsoft.com/visualstudioalm/2015/10/04/automating-azure-resource-group-deployment-using-a-service-principal-in-visual-studio-online-buildrelease-management/)). This connection is service principal based, so you need to have an AAD backing your Azure subscription and you need to have permissions to add new applications to the AAD (being an administrator or co-admin will work – there may be finer-grained RBAC roles for this, I’m not sure). However, if you don’t have an AAD backing your subscription or can’t create applications, you can manually create the cluster in your Azure subscription. Do so now if you’re going to create the cluster(s) manually (one for Test, one for Prod).

To create the release definition, go to the Release hub in VSTS and create a new (empty) Release. Select the VisualObjects build as the artifact link and set Continuous Deployment. This will cause the release to be created as soon as a build completes successfully. (If you’re on TFS, you will have to create an empty Release and then link the build in the Artifacts tab). Change the name of the release to something meaningful (I’ve called mine VisualObjects, just to be original).

Change the name of the first environment to “Test”. Edit the variables for the environment and add one called “AdminPassword” and another called “ClusterName”. Set the admin password to some password and padlock it to make it a secret. The name that you choose for the cluster is the DNS name that you’ll use to address your cluster. In my case, I’ve selected “colincluster-test” which will make the URL of my cluster “colincluster-test.eastus.cloudapp.azure.com”.

<!--kg-card-begin: html-->[![image](/assets/images/files/8419c3bc-7014-4c5e-a82d-082d37e88631.png "image")](/assets/images/files/b581edba-501a-42f1-abb1-910c6d0d292c.png)<!--kg-card-end: html-->
### Create or Update the Cluster

If you created the cluster manually, skip to the next task. If you want to create (or update) the cluster as part of the deployment, then add a new “Azure Resource Group Deployment” task to the Test environment. Set the parameters as follows:

- Azure Connection Type: Azure Resource Manager
- Azure RM Subscription: set this to the SPN connection you created from [these instructions](https://blogs.msdn.microsoft.com/visualstudioalm/2015/10/04/automating-azure-resource-group-deployment-using-a-service-principal-in-visual-studio-online-buildrelease-management/)
- Action: Create or Update Resource Group
- Resource Group: a name for the resource group
- Location: the location of your resource group
- Template: brows to the TestServiceFabricClusterTemplate.json file in the drop using the browse button (…)
- Template Parameters: brows to the TestServiceFabricClusterTemplate.parameters.json file in the drop using the browse button (…)
- Override Template Parameters: set this to -adminPassword (ConvertTo-SecureString '$(AdminPassword)' -AsPlainText -Force) –dnsName $(ClusterName)

You can override any other parameters you need to in the Override parameters setting. For now, I’m just overriding the clusterName and adminPassword parameters.

<!--kg-card-begin: html-->[![image](/assets/images/files/ce833bce-d00f-44fb-9f68-0fe7269ec8fb.png "image")](/assets/images/files/72527f6b-68ed-4604-935b-54bbccd2c276.png)<!--kg-card-end: html-->
### Replace Tokens

The Service Fabric profiles contain the cluster connection information. Since you could be creating the cluster on the fly, I’ve tokenized the connection setting in the profile files as follows:

    &lt;?xml version="1.0" encoding="utf-8"?&gt;
    &lt;PublishProfile xmlns="http://schemas.microsoft.com/2015/05/fabrictools"&gt;
      &lt;!-- ClusterConnectionParameters allows you to specify the PowerShell parameters to use when connecting to the Service Fabric cluster.
           Valid parameters are any that are accepted by the Connect-ServiceFabricCluster cmdlet.
           
           For a remote cluster, you would need to specify the appropriate parameters for that specific cluster.
             For example: &lt;ClusterConnectionParameters ConnectionEndpoint="mycluster.westus.cloudapp.azure.com:19000" /&gt;
    
           Example showing parameters for a cluster that uses certificate security:
           &lt;ClusterConnectionParameters ConnectionEndpoint="mycluster.westus.cloudapp.azure.com:19000"
                                        X509Credential="true"
                                        ServerCertThumbprint="0123456789012345678901234567890123456789"
                                        FindType="FindByThumbprint"
                                        FindValue="9876543210987654321098765432109876543210"
                                        StoreLocation="CurrentUser"
                                        StoreName="My" /&gt;
    
      --&gt;
      &lt;!-- Put in the connection to the Prod cluster here --&gt;
      &lt;ClusterConnectionParameters ConnectionEndpoint=" __ClusterName__.eastus.cloudapp.azure.com:19000" /&gt;
      &lt;ApplicationParameterFile Path="..\ApplicationParameters\TestCloud.xml" /&gt;
      &lt;UpgradeDeployment Mode="Monitored" Enabled="true"&gt;
        &lt;Parameters FailureAction="Rollback" Force="True" /&gt;
      &lt;/UpgradeDeployment&gt;
    &lt;/PublishProfile&gt;

You can see that there is a \_\_ClusterName\_\_ token (the highlighted line). You’ve already defined a value for cluster name that you used in the ARM task. Wouldn’t it be nice if you could simply replace the token called \_\_ClusterName\_\_ with the value of the variable called ClusterName? Since you’ve already installed the [Colin's ALM Corner Build and Release](https://marketplace.visualstudio.com/items?itemName=colinsalmcorner.colinsalmcorner-buildtasks) extension from the marketplace, you get the [ReplaceTokens](https://github.com/colindembovsky/cols-agent-tasks/tree/master/Tasks/ReplaceTokens) task as well, which does exactly that! Add a ReplaceTokens task and set it as follows:

<!--kg-card-begin: html-->[![image](/assets/images/files/b69d8875-9efd-46f8-9f0f-daa427cffec6.png "image")](/assets/images/files/d8d5d37e-6787-4496-8388-1981bdc033d8.png)<!--kg-card-end: html-->

**<u>IMPORTANT NOTE!</u>** The templates I’ve defined are not secured. In production, you’ll want to secure your clusters. The connection parameters then need a few more tokens like the ServerCertThumbprint and so on. You can also make these tokens that the ReplaceTokens task can substitute. Just note that if you make any of them secrets, you’ll need to specify the secret values in the Advanced section of the task.

### Deploying the App

Now that we have a cluster and we have a profile that can connect to the cluster, and we have a package ready to deploy, we can invoke the PowerShell scrip to deploy! Add a “Powershell Script” task and configure it as follows:

- Type: File Path
- Script filename: browse to the Deploy-FabricApplication.ps1 script in the drop folder (under drop/SFPackage/Scripts)
- Arguments: Set to -PublishProfileFile ../PublishProfiles/TestCloud.xml -ApplicationPackagePath ../Package

The script needs to take at least the PublishProfile path and then the ApplicationPackage path. These paths are relative to the Scripts folder, so expand Advanced and set the working folder to the Scripts directory:

<!--kg-card-begin: html-->[![image](/assets/images/files/fc9e4139-477b-425d-87f0-44383e9b8895.png "image")](/assets/images/files/7b2458d3-f552-455e-9a56-c3590e7dd56f.png)<!--kg-card-end: html-->

That’s it! You can now run the release to deploy it to the Test environment. Of course you can add other tasks (like Cloud Load Tests etc.) and approvals. Go wild.

### Changes to the OOB Deploy Script

I mentioned earlier that this technique has a snag: if the release creates the cluster (or you’ve created an empty cluster manually) then the Deploy script will fail. The reason is that the profile includes an \<UpgradeDeployment\> tag that tells the script to upgrade the app. If the app exists, the script works just fine – but if the app doesn’t exist yet, the deployment will fail. So to work around this, I modified the OOB script slightly. I just query the cluster to see if the app exists, and if it doesn’t, the script calls the Publish-NewServiceFabricApplication cmdlet instead of the Publish-UpgradedServiceFabricApplication. Here are the changed lines:

    $IsUpgrade = ($publishProfile.UpgradeDeployment -and $publishProfile.UpgradeDeployment.Enabled -and $OverrideUpgradeBehavior -ne 'VetoUpgrade') -or $OverrideUpgradeBehavior -eq 'ForceUpgrade'
    
    # check if this application exists or not
    $ManifestFilePath = "$ApplicationPackagePath\ApplicationManifest.xml"
    $manifestXml = [Xml] (Get-Content $ManifestFilePath)
    $AppTypeName = $manifestXml.ApplicationManifest.ApplicationTypeName
    $AppExists = (Get-ServiceFabricApplication | ? { $_.ApplicationTypeName -eq $AppTypeName }) -ne $null
    
    if ($IsUpgrade -and $AppExists)

Lines 1 to 185 of the script are original, (I show line 185 as the first line of this snippet). The if statement alters slightly to take the $AppExists into account – the remainder of the script is as per the OOB script.

<!--kg-card-begin: html-->[![image](/assets/images/files/5e86b754-08cc-4227-b460-f17e2b6fe0eb.png "image")](/assets/images/files/e8408851-eccb-4d52-83b2-172eeeb05876.png)<!--kg-card-end: html-->

Now that you have the Test environment, you can clone it to the Prod environment. Change the parameter values (and the template and profile paths) to make the prod-specific and you’re done! One more tip: if you change the release name format (under the general tab) to

<!--kg-card-begin: html--><font face="Courier New">$(Build.BuildNumber)-$(rev:r)</font><!--kg-card-end: html-->

then you’ll get the build number as part of the release number.

Here you can see my cluster with the Application Version matching the build number:

<!--kg-card-begin: html-->[![image](/assets/images/files/f51dbe3a-f2c3-4eca-9e5b-f0d548557f88.png "image")](/assets/images/files/898c74ec-07c4-41f6-9fb5-cbaddbf83957.png)<!--kg-card-end: html-->

Sweet! Now I can tell which build was used for my application right from my cluster!

### See the Pipeline in Action

A fun demo to do is to deploy the app and then open up the VisualObjects url – that will be at clustername.eastus.cloudapp.azure.com:8082/VisualObjects (where clustername is the name of your cluster). When you see the bouncing triangles.

Then you can edit

<!--kg-card-begin: html--><font face="Courier New">src/VisualObjects/VisualObjects.ActorService/VisualObjectActor.cs</font><!--kg-card-end: html-->

in Visual Studio or in the Code hub in VSTS. Look around line 50 for

<!--kg-card-begin: html--><font face="Courier New">visualObject.Move(false);</font><!--kg-card-end: html-->

and change it to

<!--kg-card-begin: html--><font face="Courier New">visualObject.Move(true)</font><!--kg-card-end: html-->

. This will cause the triangle to start rotating. Commit the change and push it to trigger the build and the release. Then monitor the Service Fabric UI to see the upgrade trigger (from the release) and watch the triangles to see how they are upgraded in the Service Fabric rolling upgrade.

## Conclusion

Service Fabric is awesome – and creating a build/release pipeline for Service Fabric apps in VSTS is a snap thanks to an amazing build/release engine – and some cool custom build tasks!

Happy releasing!

