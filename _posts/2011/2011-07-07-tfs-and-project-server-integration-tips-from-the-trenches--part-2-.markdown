---
layout: post
title: 'TFS and Project Server Integration: Tips from the Trenches (Part 2)'
date: '2011-07-07 20:14:00'
tags:
- alm
---

## Links to this series:

- [Part 1 – Considerations](http://colinsalmcorner.blogspot.com/2011/07/tfs-and-project-server-integration-tips.html)
- [Part 3 – Configuration](http://colinsalmcorner.blogspot.com/2011/07/tfs-and-project-server-integration-tips_165.html)
- [Part 4 – Synchronizing Hierarchies from TFS](http://colinsalmcorner.blogspot.com/2011/07/tfs-and-project-server-integration-tips_6118.html)

## Part 2: Setup and Configuration

The best way to do this is to follow the instructions from the “Configuration Quick Reference” page in the Integration help file. In this post I’ll add some summaries and screen shots to augment the manual a bit.

## Required Accounts

1. An account that will be used to configure the integration _globally_ – or at least at the Project Collection\<-\>PWA level. (I used the **tfssetup** account that was used to install and configure TFS). (For the rest of this post, this account will be referred to as tfssetup)
2. Accounts that will be used to manage mappings _per project_ – at the TeamProject\<-\>Enterprise Project level (these can be Project Admins or Team Leads on the TFS side). (This can also be the tfssetup account if you want – for this post, I will use the **colind** account)
3. The TFS Service account ( **tfsservice** for this post).

## Configuring Permissions on TFS

1. Make sure the global config account ( **tfssetup** ) is a in the **Team Foundation Administrator** group
2. Open the TFS Admin Console and click on the Application Tier node
3. Click on the “Group Membership” link in the Application Tier Summary section
4. Add the user (tfssetup) to the Team Foundation Administrator group
<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-KST5sPIGgu0/ThWU-K7stHI/AAAAAAAAARQ/mQgvu4Xcid0/image_thumb4.png?imgmax=800 "image")](http://lh4.ggpht.com/-hCEnV4A9a3k/ThWU89XjpzI/AAAAAAAAARM/m1GGl6RGoso/s1600-h/image10.png)<!--kg-card-end: html-->
1. Make sure each account that will be managing mapping at the project level ( **colind** ) has “Administer Project Server integration” permissions.
2. Open the TFS Admin Console and click on the “Team Project Collections” node
3. Click on the Project Collection you want to configure
4. Click on “Administer Security”
5. Select the group that the user is in (or add the user explicitly) and make sure the “Administer Project Server integration” permission is allowed
<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-_wSLbHjFIfw/ThWVAKJGH0I/AAAAAAAAARY/F5Xb4pkFpkE/image_thumb8.png?imgmax=800 "image")](http://lh3.ggpht.com/-if6zP0elJ1c/ThWU-0dqjDI/AAAAAAAAARU/7a856iiaBGI/s1600-h/image18.png)<!--kg-card-end: html-->
1. Make sure the resources you are going to use on the project are part of some group on the TFS Team Project that you will be mapping to Project Server (usually just place them into the Contributors group).

## Configuring Permissions on Project Server 2010

1. Configure service account permissions on the Project Server Application in Sharepoint Central Admin
2. Open the Sharepoint central admin console and click on the “Manage Service Applications” link
<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-6dr6IkYLDYI/ThWVB9kyqDI/AAAAAAAAARg/WEEzu2TNlVA/image_thumb11.png?imgmax=800 "image")](http://lh6.ggpht.com/-8EP177a3cDc/ThWVAxu5NcI/AAAAAAAAARc/0uNQBSwX3Ls/s1600-h/image23.png)<!--kg-card-end: html-->
1. Look for the “Project Server Service Application” and click next to the name (don’t click the hyperlink – you just want to select the row, not follow the link – see the red ‘x’ in the picture below). This highlights the row and activates some buttons in the ribbon at the top.
<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-HJ9-69v0WDc/ThWVDqakRvI/AAAAAAAAARo/wSS7Uv7cXUc/image_thumb13.png?imgmax=800 "image")](http://lh5.ggpht.com/-FR4urAO2qWU/ThWVCTKKWPI/AAAAAAAAARk/q5GiJUdkTaQ/s1600-h/image27.png)<!--kg-card-end: html-->
1. Click on the “Permissions” button in the ribbon.
2. Add the TFSService account and make sure it has full control checked in the permissions section.
<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-STa_dLxbT00/ThWVE_NgQzI/AAAAAAAAARw/XNa_flU9rXo/image_thumb15.png?imgmax=800 "image")](http://lh5.ggpht.com/-RGPNdXyxxHc/ThWVEI5LbcI/AAAAAAAAARs/OZTiF6ULbbo/s1600-h/image31.png)<!--kg-card-end: html-->
## Configuring Permissions in PWA

To configure the integration, you’ll need to create 2 user accounts on the PWA – one for the TFSService account and one for the account you’ll use to register the PWA (TFSSetup). You can add them to the Administrator group on PWA or assign the accounts the minimum permissions as described in the Integration help file.

Each user that is going to have tasks must be created as an enterprise resource. Make sure the display name matches the display name for the user in Active Directory – and it’s recommended that you let Project server sync users from Active Directory.

The resources will need to be part of the Team Members group in Project Server (or have the same permission set).

Once you’ve created the Enterprise Project, make sure that you select “Build Team” and assign the enterprise resources to the project.

So now you’ve got the permissions set and the components installed – next we’ll configure the bits.

