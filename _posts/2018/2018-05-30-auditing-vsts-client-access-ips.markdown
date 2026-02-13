---
layout: post
title: Auditing VSTS Client Access IPs
date: '2018-05-30 20:27:41'
tags:
- tfsconfig
---

Visual Studio Team Services (VSTS) is a cloud platform. That means it's publicly accessible from anywhere - at least, by default. However, Enterprises that are moving from TFS to VSTS may want to ensure that VSTS is only accessed from a corporate network or some white-list of IPs.

To enable conditional access to VSTS, you'll have to have an Azure Active Directory (AAD) backed VSTS account. The conditional access is configured on AAD, not on VSTS itself, since that's where the authentication is performed. For instructions on how to do this, read this [MSDN article](https://docs.microsoft.com/en-us/vsts/accounts/manage-conditional-access?view=vsts).

### Audit Access

However, this may be a bit heavy-handed. Perhaps you just want to audit access that isn't from a white-list of IPs, instead of blocking access totally. If you're an administrator, you may have come across the Usage page in VSTS. To get there, navigate to the landing page for your VSTS account, click the gear icon and select Usage:

<!--kg-card-begin: html-->[![image](/assets/images/files/44cd5992-8b35-4e70-abdc-93866be34952.png "image")](/assets/images/files/00fe0946-f583-4439-8c81-73f320aa5dd1.png)<!--kg-card-end: html-->

This page will show you all <u>your</u> access to VSTS. To see the IPs, you have to add the "IP Address" column in the column options:

<!--kg-card-begin: html-->[![image](/assets/images/files/84d10ee7-8496-40c5-9a7f-a19f4db05107.png "image")](/assets/images/files/9e0404a1-2f29-4152-82c0-d06137b0dfff.png)<!--kg-card-end: html-->

Nice, but what about other users? To get that you have to use the VSTS REST API.

## Dumping an Exception Report with PowerShell

There is an (undocumented) endpoint for accessing user access. It's [https://\<your account\>.visualstudio.com/\_apis/Utilization/UsageSummary](https://<your account>.visualstudio.com/_apis/Utilization/UsageSummary) with a whole string of query parameters. And since it's a REST call, you'll need to be authenticated so you'll have to supply a header with a base-64 encoded Personal Access Token (PAT).

Using PowerShell, you can make a call to the endpoint and then filter results where the IP is not in the white-list of IPs. Fortunately for you, I've made a gist for you, which you can access [here](https://gist.github.com/colindembovsky/e4f5d67ab807914cad5af1e5752e00de). When you call the script, just pass in your account name, your PAT, the start and end date and the white-list of IPs. Any access from an IP not in this list is dumped to a CSV.

<!--kg-card-begin: html-->[![image](/assets/images/files/d073731e-f3d1-417e-8be5-bd2408a325f3.png "image")](/assets/images/files/cac7d178-31c6-4339-8456-501e15cb9f4a.png)<!--kg-card-end: html-->
### Limitations

There are some limitations to the API:

1. You'll need to be a Project Collection or Account admin to make this call (since there's no documentation, I'm guessing here).
2. You can only go back 28 days, so if you need this as an official exception report you'll have to schedule the run.

## Conclusion

VSTS knows the client IP for any access. Using the API, you can dump a list of access events that are not from a white-list of IPs.

Happy auditing!

