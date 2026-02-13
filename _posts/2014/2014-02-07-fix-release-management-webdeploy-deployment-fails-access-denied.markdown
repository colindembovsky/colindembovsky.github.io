---
layout: post
title: 'Fix: Release Management WebDeploy Deployment Fails: Access Denied'
date: '2014-02-07 18:17:00'
tags:
- releasemanagement
---

If you’re using WebDeploy and Release Management ([as you should](http://www.colinsalmcorner.com/2013/11/webdeploy-and-release-management.html) to release Web Applications) you may hit the following error:

<!--kg-card-begin: html--><font size="2" face="Courier New">Info: Adding sitemanifest (sitemanifest).<br>Info: Creating application (Default Web Site/MyWebsite)<br>Error: An error occurred when reading the IIS Configuration File 'MACHINE/REDIRECTION'. The identity performing the operation was 'DOMAIN\tfsservice'.<br>Error: Filename: \\?\C:\Windows\system32\inetsrv\config\redirection.config<br>Error: Cannot read configuration file due to insufficient permissions</font><!--kg-card-end: html-->

Seems that the WebDeploy command can’t access some files in c:\Windows\system32\inetsrv. It may be the irmsdeploy.exe MSDeploy wrapper that I’m using for doing WebDeploy in Release Management (see my [post about how to do this](http://www.colinsalmcorner.com/2013/11/webdeploy-and-release-management.html)), since logging into the machine and running the webdeploy.cmd file manually works just fine.

## The Resolution

You have to add permissions for the release management agent identity to the folder, but this is a folder who’s owner identity is TrustedInstaller – meaning you have to change the owner to yourself first.

- Right click the insetsrv folder in c:\windows\system32 and select Properties.
- Click on the Security tab and click the “Advanced” button:
<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-SMFFZcKdfi0/UvSWhMDKxUI/AAAAAAAABOw/d86F02LLIaE/image_thumb.png?imgmax=800 "image")](http://lh6.ggpht.com/-SXLi5YiFJJ8/UvSWgtofeyI/AAAAAAAABOo/m7-YRNPhdvg/s1600-h/image%25255B2%25255D.png)<!--kg-card-end: html-->
- Click on the owner tab and then on the Edit button:
<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-QNJhFwsmd9M/UvSWiNpdQdI/AAAAAAAABPA/X--C0nBBLEo/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-hx6-XXpFqHI/UvSWhj12lvI/AAAAAAAABO4/eIvZ_GGn0BU/s1600-h/image%25255B5%25255D.png)<!--kg-card-end: html-->
- Select yourself (I logged in as TfsSetup which is in the local admin group on this machine), check “Replace owner on subcontainers and objects” checkbox and click “OK”:
<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-XrHxyKXxteU/UvSWjSVE48I/AAAAAAAABPQ/phXqt3yXlV4/image_thumb%25255B2%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-2LshUZF4El4/UvSWi59mevI/AAAAAAAABPI/pZdmwSbM9TU/s1600-h/image%25255B8%25255D.png)<!--kg-card-end: html-->
- Close all the dialogs and then right-click the inetsrv folder again and click Properties. Now you can allow read access to the Release Management agent identity to this folder.

Once you’ve changed the permissions, you will need to reboot the machine. After the reboot, the WebDeploy through Release Management should work without a hitch.

Happy releasing!

