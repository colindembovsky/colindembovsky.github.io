---
layout: post
title: WebDeploy and Release Management – The Proper Way
date: '2013-11-29 23:50:00'
tags:
- releasemanagement
---

I’ve just completed recording a Release Management webcast (more on Imaginet’s Visual Studio webcasts [here](http://www.imaginet.com/events)). While doing the webcast, I wanted to show how you can use tokens which Release Management can substitute during the Release Workflow. Brian Keller suggests a .token file (basically an exact copy of your web.config file except that you use tokens instead of values) in his [Release Management hands on lab](http://download.microsoft.com/download/B/C/8/BC8558E1-192E-4286-B3B0-320A8B7CE49D/Embracing%20Continuous%20Delivery%20with%20Release%20Management%20for%20Visual%20Studio%202013.docx), but I hate having to keep 2 copies of the same file around.

Of course, being a huge fan of Web Deploy, I could use config transforms. The problem with mixing config transforms and Release Management is that you’d have to have a configuration per environment in your solution, and you’d end up having to create _n_ number of web deploy packages where _n_ is the number of Release Management stages you have. So if you had Dev, QA and Prod stages, you’d have to add at least one configuration to your solution so that you’d have Debug (for Dev), QA (for the QA environment) and Release (for Prod). Technically you wouldn’t be deploying the same package to each environment, even though they could be built at the same time from the same source files.

I bingled a bit and found two posts that looked useful. The [first was about tokenization](http://support.inreleasesoftware.com/entries/21448302-Tokenization-of-configuration-files), and had the downside is that you’re still doing an xcopy deployment rather than a web deploy package deployment, and you have to do some rather nasty gymnastics with the build parameters in order get it to work. The [second](http://support.inreleasesoftware.com/entries/21487316-InRelease-with-Web-Deploy) was a lot cleaner, except for the fact that you have to know the folder on the server where the website ends up after the web deploy command, since the token replacement is done _after_ the invocation of the web deploy Tool.

I was convinced there was a cleaner solution, and I managed to come up with one. Basically, we use Web Deploy Parameters to tokenize the web.config file, and then do the token replacement before invoking Web Deploy.

## Parameters.xml

Web Deploy lets you define parameters in a file called Parameters.xml. If there isn’t one in your project (alongside the web.config) then it creates a default one during publishing, so normally you don’t see it at all.

Let’s imagine that you have the following web.config snippet:

    <connectionstrings>
      <add name="FabrikamFiber-Express" connectionstring="somestring" providername="System.Data.SqlClient">
    </add></connectionstrings>
    <appsettings>
      <add key="webpages:Version" value="2.0.0.0">
      <add key="PreserveLoginUrl" value="true">
      <add key="ClientValidationEnabled" value="true">
      <add key="UnobtrusiveJavaScriptEnabled" value="true">
      <add key="DemoEnv" value="Development">
    </add></add></add></add></add></appsettings>

There’s a connection string and an appSetting key that we want to tokenize. Right click the project and add a new xml file called “Parameters.xml”. Right click the file, select Properties and set the “Build Action” to None to make sure this file doesn’t end up deployed to your website. Now we add the following xml:

    <!--?xml version="1.0" encoding="utf-8" ?-->
    <parameters>
      <parameter name="DemoEnv" description="Please enter the name of the Environment" defaultvalue=" __EnvironmentName__" tags="">
        <parameterentry kind="XmlFile" scope="\\web.config$" match="/configuration/appSettings/add[@key='DemoEnv']/@value">
      </parameterentry></parameter>
    </parameters>

We create a parameter with a name, description and default value. The format of the default value is important – it needs to be pre- and post-fixed with double underscore “\_\_” since this is the token format for Release Management. We then specify a “kind” of XmlFile, set web.config as the scope and specify an XPath to find the parameter in the web.config file.

(You’ll notice that we don’t need to specify any parameter for the connection string, since that will be tokenized in the publish profile)

## Publish Profile

Right-click your web project and select “Publish” to create (or edit) a publish profile. Expand the dropdown and select &nbsp;to create a new profile. I named mine “Release”. Click Next.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-L2-25D-E9Zs/UpibUzdojhI/AAAAAAAABHY/djCwst29Ao4/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-BKy-yRAbHXk/UpibT9ZeS_I/AAAAAAAABHQ/-fIW3W4iOOI/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

On the Connection page, select “Web Deploy Package” as the publish method and enter a name for the package location. Typically this is (name of your project).zip. For Site name, enter “\_\_SiteName\_\_” to create a Release Management token for your site name. Click Next.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-ky2u8_fgRjg/UpibWc3G-eI/AAAAAAAABHo/9fKAZ66AJ6I/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-dr2kUo3r4qk/UpibVv_h09I/AAAAAAAABHg/0-XrR3y2Y6c/s1600-h/image%25255B7%25255D.png)<!--kg-card-end: html-->

On the Settings page, select your configuration (I selected Release, which applies and config transforms in the Web.Release.config file in my solution, such as removing the Debug attribute from the &nbsp;element). For each connection string you have, instead of entering in a real connection, again enter a Release Management token – I entered “\_\_FabrikamFiber-Express-Connection\_\_”.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-n6Dy-Hh0AP0/UpibYFHx2sI/AAAAAAAABH4/5LgEgrjYwNU/image_thumb%25255B5%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-bLD-gkoVE7w/UpibXVfLJdI/AAAAAAAABHw/H7y4q5xfpr8/s1600-h/image%25255B11%25255D.png)<!--kg-card-end: html-->

Click Close and save the profile. The profile appears under the Properties\PublishProfiles folder of your web project.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-4X-gvNm-JSU/UpibZD86GgI/AAAAAAAABII/twPi65DvaUQ/image_thumb%25255B7%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-7m6izo5bi-Q/UpibYY9SZUI/AAAAAAAABIA/zxRvh9PaPKQ/s1600-h/image%25255B15%25255D.png)<!--kg-card-end: html-->

Now check your solution into source control.

If you do actually publish, you’ll see a SetParameters.xml file alongside the web deploy zip file. The contents of the file should be something like this:

    <!--?xml version="1.0" encoding="utf-8"?-->
    <parameters>
      <setparameter name="IIS Web Application Name" value=" __SiteName__">
      <setparameter name="DemoEnv" value=" __EnvironmentName__">
      <setparameter name="FabrikamFiber-Express-Web.config Connection String" value=" __FabrikamFiber-Express-Connection__">
      <setparameter name="FabrikamFiber.DAL.Data.FabrikamFiberWebContext-Web.config Connection String" value="FabrikamFiber.DAL.Data.FabrikamFiberWebContext_ConnectionString">
    </setparameter></setparameter></setparameter></setparameter></parameters>

You can see that there are 3 Release Management tokens – SiteName, EnvironmentName and FabrikamFiber-Express-Connection. These are the tokens we’ll replace when creating a release component for this website.

## Building the Package

You can now create a build – make sure you use the ReleaseDefaultTemplate11.1.xaml (from the Release Management bin folder). Specify any arguments you want to as usual, but make sure you have this in the MSBuild arguments:

<!--kg-card-begin: html--><font size="3" face="Courier New">/p:DeployOnBuild=true;PublishProfile=Release</font><!--kg-card-end: html-->

That will instruct web deploy to create the package when we build using the settings of the Release profile. I recommend that you set “Release to Build” to false just to make sure your build is producing the correct artifacts.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-X8uDXRV0rCU/UpibaqIWiiI/AAAAAAAABIY/Uxpr3yprHQ4/image_thumb%25255B9%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-uKhtFlTnzyo/UpibZ4qFJxI/AAAAAAAABIQ/y2ZsO6HSoQM/s1600-h/image%25255B19%25255D.png)<!--kg-card-end: html-->

After running the build, you should see the following in the \_PublishedWebSites folder of your drop:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-kUwemv74Z64/UpibcBVq8oI/AAAAAAAABIo/U21i-1S2HTU/image_thumb%25255B11%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-gHn2eC9W3fg/UpibbeduqMI/AAAAAAAABIg/zItSLhdGzqQ/s1600-h/image%25255B23%25255D.png)<!--kg-card-end: html-->

If you open the SetParameters.xml file (the one highlighted above) then you should see your Release Management tokens.

## Create a Web Deploy Tool in Release Management

Download the InRelease MS Deploy wrapper from [here](http://support.inreleasesoftware.com/attachments/token/kfvabql0jv4omje/?name=irmsdeploy.exe). This is a simple exe that wraps the call to Web deploy so that any errors are reported in a way that Release Management understands. Let’s then go to Release Management-\>Inventory-\>Tools and click on New to create a new Tool:

Enter the following parameters:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-27wu5FbvcGg/UpibdWVY3OI/AAAAAAAABI4/B4GX9Y_xoh8/image_thumb%25255B13%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-clawD7s-38g/UpibcsqSGNI/AAAAAAAABIw/OZr35qPHyEg/s1600-h/image%25255B27%25255D.png)<!--kg-card-end: html-->

You can see that I made a new parameter called “WebAppName” to make this a generic tool.

At the bottom under Resources, click the “Add” button and import the irmsdeploy.exe that you downloaded.

**<u>Update: 2013-12-02</u>** When running through this “demo” again on my VM checkpoint, the WebDeploy custom tool kept failing with an “Unable to find file” exception. After trying this several times and tearing my hair out in chunks, I thought I would make sure this wasn’t a security issue – turns out, it was exactly that. Once you’ve downloaded the irmsdeploy.exe file, make sure you right-click it, select properties and “Unblock” it (see the below screenshot).

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-ZtXq92R57no/UpzblJm-d8I/AAAAAAAABLE/qWOijdmN0sQ/image_thumb.png?imgmax=800 "image")](http://lh3.ggpht.com/-DUYVtKdFZlE/UpzbinXEAbI/AAAAAAAABK8/NPaqH1fm-lQ/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

## Create a Component

Once you’ve set up your Release Path, you’ll be able to define the release workflow for each stage. You’re going to need to create a component for your website in order to deploy it. Navigate to “Configure Apps-\>Components” and click New. Enter a name (and optional description) and then enter the following on the Source tab:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/--wVySNRIExI/UpibfIyZfVI/AAAAAAAABJI/kGVuQ0pxJxs/image_thumb%25255B15%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-cuRJ-5r4AMQ/UpibeaE8hdI/AAAAAAAABJA/ODMZaTPHUN8/s1600-h/image%25255B31%25255D.png)<!--kg-card-end: html-->

The “Path to Package” is the path that contains the web deploy package.

Now click on the “Deployment” tab:.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-5JOJUUe-Mzg/Upibgen3uCI/AAAAAAAABJY/Ceo8wP7RWqs/image_thumb%25255B17%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-OvMisXbj-Ig/UpibfiIJmOI/AAAAAAAABJQ/sYX_w3m0kJ0/s1600-h/image%25255B35%25255D.png)<!--kg-card-end: html-->

Select WebDeploy from the Tool dropdown – that will automatically create the WebAppName parameter for this component.

Finally, move to the “Configuration Variables” tab:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-Z3Bb_yMFpFo/UpibhtVqlMI/AAAAAAAABJo/juQ1WIBOCrs/image_thumb%25255B26%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-9HZdDKHIzNc/UpibhDM1GsI/AAAAAAAABJg/866sYFe6DEk/s1600-h/image%25255B48%25255D.png)<!--kg-card-end: html-->

Change the “Variable Replacement Mode” to “Before Installation” and set the “File Extension Filter” to “\*.SetParameters.xml”. Now add all of your parameters (these are the tokens that are in your SetParameters.xml file after the build). You can type descriptions too.

## Specifying Values in a Release Template

We’re finally ready to use the component inside a Release Template. Create a new template for a release path, and inside a Server activity drop your component. When you expand it, you’ll see that you can specify values for the parameters. Here are screenshots of my Dev and QA components:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-RnR2KZA1OeA/Upibi_mtMeI/AAAAAAAABJ4/SAtDRZ6pM7E/image_thumb%25255B39%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-UD7iPt1-3H0/UpibiAJaFvI/AAAAAAAABJs/TzD6LX0QVOs/s1600-h/image%25255B65%25255D.png)<!--kg-card-end: html--><!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-rBhwrP4AaGQ/Upibj5XAWUI/AAAAAAAABKI/c7xsUGtqZZ0/image_thumb%25255B42%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-0T4pIX50Wak/UpibjUMIWVI/AAAAAAAABJ8/F5FZ2DGVBVA/s1600-h/image%25255B68%25255D.png)<!--kg-card-end: html-->

The best part is, not only can you use these parameters in Release Management, but if you import this web deploy package directly in IIS, you get prompted to supply values for the parameters too:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-YhinJ2iGZwI/UpibmqkG-eI/AAAAAAAABKY/AzlHE2rYj9I/image_thumb%25255B44%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-itWALleab10/UpiblWgZXqI/AAAAAAAABKQ/okA7XG__Uuk/s1600-h/image%25255B72%25255D.png)<!--kg-card-end: html-->

Happy Releasing!

