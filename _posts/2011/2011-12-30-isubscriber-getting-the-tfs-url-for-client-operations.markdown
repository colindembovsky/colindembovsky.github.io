---
layout: post
title: 'ISubscriber: Getting the TFS Url for Client Operations'
date: '2011-12-30 05:03:00'
tags:
- alm
---

Responding to TFS events can be done in (at least) 2 ways: create a SOAP webservice and register with [bissubscribe](http://msdn.microsoft.com/en-us/magazine/cc507647.aspx) (this works in a “client” fashion) or implement the [ISubscriber interface](http://msdn.microsoft.com/en-us/library/microsoft.teamfoundation.framework.server.isubscriber.aspx) (in the [Microsoft.TeamFoundation.Framework.Server](http://msdn.microsoft.com/en-us/library/microsoft.teamfoundation.framework.server.aspx) namespace).

The advantage to the ISubscriber interface implementation is that the plugin can be installed on the TFS server and can also be used to allow or disallow a change (such as a check-in policy). One disadvantage of ISubscribers is that you only get access to a [TeamFoundationRequestContext](http://msdn.microsoft.com/en-us/library/microsoft.teamfoundation.framework.server.teamfoundationrequestcontext.aspx) object, not a TfsTeamProjectCollection object. This can limit what operations you can perform.

I was working with an ISubscriber and wanted to update a Global List, so I needed a reference to the [WorkItemStore](http://msdn.microsoft.com/en-us/library/microsoft.teamfoundation.workitemtracking.client.workitemstore.aspx) object. Unfortunately, there was no obvious way to do this from the TeamFoundationRequestContext. I could have hard-coded a TFS url into a config file, but this felt like a cop out to me. So I dug around a little bit more and came up with a solution: the TeamFoundationLocationService (see [this article](http://msdn.microsoft.com/en-us/library/ms252473.aspx) about TFS services).

## The TeamFoundationLocationService

You can easily get the TeamFoundationLocationService from the TeamFoundationRequestContext. You can then query the location service to get the url of the TFS server and collection. Once you have that, you can then instantiate a TfsTeamProjectCollection object and use that to get the WorkItemStore. Here’s the code:

    private Uri GetTFSUri(TeamFoundationRequestContext requestContext)<br>{<br> var locationService = requestContext.GetService<teamfoundationlocationservice>();<br> return new Uri(locationService.ServerAccessMapping.AccessPoint + "/" + requestContext.ServiceHost.Name);<br>}<br><br>private WorkItemStore GetWorkItemStore(TeamFoundationRequestContext requestContext)<br>{<br> var uri = GetTFSUri(requestContext);<br> var tpc = TfsTeamProjectCollectionFactory.GetTeamProjectCollection(uri);<br> return tpc.GetService<workitemstore>();<br>}<br></workitemstore></teamfoundationlocationservice>

Happy subscribing!

