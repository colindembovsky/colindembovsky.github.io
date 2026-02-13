---
layout: post
title: 'Lab management: The Dreaded “Unknown Error: 0x8033811e”'
date: '2011-04-14 17:42:00'
tags:
- labmanagement
---

I have been setting up a TFS for demos and web training. Since TFS installation is now really easy, I got it up quickly and configured Lab Management. Everything was plain sailing until I tried to deploy a stored environment. The SCVMM job failed and provided this rather unhelpful message (where host.com is my host server):

_A Hardware Management error has occurred trying to contact server host.com (Unknown error 0x8033811e). Check that WinRM is installed and running on the server host.com._

Well I did check WinRM – no problems there. So I started to search for other solutions – found one about ip listening ports and a whole bunch of other nothing. After sifting through some obscure sites, I managed to narrow the solution to one of two things:

1. BITS port conflicts
2. Delegation issues

## Changing the BITS Port

By default, BITS uses port 443 to transfer files – so if you have anything using SSL or firewalls blocking the port, you’ll have issues with BITS. I ignored this at first since the SCVMM job seemed to get past copying the VHD over BITS. Anyway, this turned out to be the solution to my problem. Here’s how you fix it:

1. Open the registry of the VMM server machine
2. Find HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Microsoft System Center Virtual Machine Manager Server\Settings
3. Edit the BITSTcpPort to something other than 443 (I used 8050)
4. Restart the “Virtual Machine Manager” service

## Fixing Delegation Issues

The other possibility is delegation issues. Follow these steps:

1. Go to the Domain Controller and open “Active Directory Users and Computers”
2. In the Computers node, find the host machine and open its properties
3. Go to the Delegation tab and check “Trust this computer for delegation to specified services only”
4. Select “Use any authentication protocol”
5. In the list of services, click “Add” and find the VMM server computer
6. When you click OK, a list of services is shown – find the “cifs” service, select it and click ok

Happy labbing!

