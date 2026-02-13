---
layout: post
title: WebDeploy Gets Even More Awesome – Profile Specific Transforms
date: '2014-04-23 16:37:00'
tags:
- releasemanagement
---

I love WebDeploy – I have ever since I read Scott Hanselman’s post [“Web Deploy Made Awesome: If You’re Using XCopy, You’re Doing It Wrong”](http://www.hanselman.com/blog/WebDeploymentMadeAwesomeIfYoureUsingXCopyYoureDoingItWrong.aspx). Whenever I’m helping teams that build web applications improve their ALM processes, invariable I end up moving them onto Web Deploy. Not only is it an easier and cleaner way to deploy, but you get the bonus of being able to manage configuration files (Web.config) in your project.

## Config as Code

Maturing in ALM means that you need to head towards continuous deployment. You need to be able to deploy your application with a single click of a button. However, one challenge with this is managing configurations. How do you manage Web.config files in dev, UAT and Production environments? Do you find yourself copying config files out the way before you deploy? That’s BAD. You should be managing your configs as if they were code. They need to be source controlled.

Fair enough, I hear you say. So you’ll just copy all your config files from all your servers into a folder and manage them from there. Better – but still lots of pain. A far better approach is to use config transforms.

Config transforms came into web applications in VS 2010. With the latest release of VS 2013, not only are they also available for web sites (NOTE: please don’t use websites – always use web applications!) but you can new preview your transforms from VS and there’s now support for profile-specific transforms.

Let’s have a look at a config transform in a newly created MVC web application. You can see that the Web.config file expands to show 2 other files – one per project configuration:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-lJ00JDqtfSc/U1dtlA39wWI/AAAAAAAABV8/cTDXA9jNxVM/image_thumb.png?imgmax=800 "image")](http://lh6.ggpht.com/-YGiHKqsIuZ8/U1dtkN4GlOI/AAAAAAAABV0/b3mj46J2nIU/s1600-h/image%25255B2%25255D.png)<!--kg-card-end: html-->

If you add project configurations, then you can right-click the Web.config and select “Add config Transforms” and VS will create a new transform file for you.

Let’s have a look at the Web.Release.config (I’ve removed the comments that are auto-generated):

    <configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
      <system.web>
        <compilation xdt:transform="RemoveAttributes(debug)">
      </compilation></system.web>
    </configuration>

You can see that in the &nbsp;tag there’s an xdt:Transform to remove the debug attribute. When applied, the transform uses the structure of the transform file to find the correct element and perform the necessary transforms – whether it’s an insert, a remove or some other transform. You can read about the syntax for transforms [here](http://msdn.microsoft.com/en-us/library/dd465326(v=vs.110).aspx). Note that when you debug out of VS, your application will use the Web.config file irrespective of if you’re running in Debug or Release – the transforms only happen when you publish or package your web app.

This is all existing “2012” stuff. What’s new in the latest release?

## Preview and Profile Specific Transforms

You can now preview your transform by right-clicking on the transform file (Web.Release.config, for example) and selecting “Preview Transform”. This immediately shows a diff – the original Web.config on the left and the transformed config on the right:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-8nl4r4oTsmA/U1dtn3KHksI/AAAAAAAABWM/r2Awo15BsdI/image_thumb%25255B2%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-FRvfdnnlHAU/U1dtmdB-kTI/AAAAAAAABWE/f8OzAWgYOP4/s1600-h/image%25255B6%25255D.png)<!--kg-card-end: html-->

Now you can debug your transforms!

So let’s imagine you have a UAT environment. You want to publish your site, so you create a new project configuration called UAT. You then copy the release transforms and put in your UAT connection strings and so on. After doing some testing in UAT, you realize that you actually need to debug, so you’ll have to update your Web.UAT.config file again, or create a UAT-DEBUG project configuration. Soon you’ll get lots of project configurations, and that’s just a mess.

However, you can now create a profile-specific transform on top of the project configuration transforms! Let’s create a publish profile. I’ll right-click the web project and select “Publish”. I’ll create a new profile called “UAT-Debug” and give it whatever settings I need to. On the “Configuration” page, I’ll change the configuration from Release to Debug.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-CNFnS-2x-tM/U1dtpdKZjwI/AAAAAAAABWc/VmX_sjROfCU/image_thumb%25255B4%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-Ed1Le7fHzUI/U1dtokAE42I/AAAAAAAABWU/z01rc2Sh_4Q/s1600-h/image%25255B10%25255D.png)<!--kg-card-end: html-->

Then I’ll click on “Close” to save the profile without publishing. That will create a pubxml file for me. I’ll repeat the process and create a publish profile called “UAT-Release”. Looking under the Properties of my project I’ll see the pubxml files:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-bQ7HqTRb49w/U1dtq7Yb5TI/AAAAAAAABWs/380jDzBBqs0/image_thumb%25255B6%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-TMI7lv6iMsE/U1dtqMIv9fI/AAAAAAAABWk/hV7eh3j_V5g/s1600-h/image%25255B16%25255D.png)<!--kg-card-end: html-->

Now I can right-click a pubxml and select “Add Config Transform” – which creates a new transform for me that’s tied to this publish profile.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-4a-PCyFWa5M/U1dtsp7CUWI/AAAAAAAABW8/btc9n2ZAEuA/image_thumb%25255B7%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-73XD-8jNbg8/U1dtrpPahzI/AAAAAAAABW0/UlSYjbBx8Gs/s1600-h/image%25255B19%25255D.png)<!--kg-card-end: html-->

I’ll create one for each profile. You’ll see that I now have 4 transforms (but still only 2 project configurations – hooray!)

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-9zZ2EkJIfMU/U1dtt7iVk8I/AAAAAAAABXM/hPFKWV-21ak/image_thumb%25255B8%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-zHhXxUy9JCo/U1dttULnqJI/AAAAAAAABXE/_UZ1iOC2OBo/s1600-h/image%25255B22%25255D.png)<!--kg-card-end: html-->

Let’s insert an attribute in both of the new config files:

    <appsettings>
      <add key="Profile" value="UAT-debug" xdt:transform="Insert">
    </add></appsettings>

I’ve also removed the debug attribute from the compilation tag on the UAT-Release config and removed that transform from the debug one:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-uKlJh1sJ63A/U1dtvvQRWSI/AAAAAAAABXc/K16S2zGhATg/image_thumb%25255B10%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-EF7G_5Bhsek/U1dtuykRrZI/AAAAAAAABXU/XeErJsRZPCs/s1600-h/image%25255B26%25255D.png)<!--kg-card-end: html-->

Now when I right-click Web.UAT-Release.config and select “Preview Transform” I can see the diff. Note that the right-hand file tells me that it has actually applied 2 transforms: first the Release transform (since the profile specifies Release for the project configuration) and secondly the Web.UAT-Release.config transforms themselves.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-iYnIq-p7JFI/U1dty8jNAJI/AAAAAAAABXs/_y_vP11HLBo/image_thumb%25255B12%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-iVpv87hfRIk/U1dtxAZ8FJI/AAAAAAAABXk/4105KU36sOA/s1600-h/image%25255B30%25255D.png)<!--kg-card-end: html-->

Voila! I can now maintain 2 project configurations (Debug and Release) and have a publish profile and transform config per project config and environment. Awesome!

If you’re using Release Management, don’t forget to check out my post about [how to use config transforms for parameterizing releases](http://www.colinsalmcorner.com/2013/11/webdeploy-and-release-management.html)!

Happy transforming!

