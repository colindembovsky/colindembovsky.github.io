---
layout: post
title: Azure Web Apps, Kudu Console and TcpPing
date: '2015-08-07 03:32:53'
tags:
- cloud
---

I was working with a customer recently that put a website into Azure Web Apps. This site needed to connect to their backend databases (which they couldn’t move to Azure because legacy systems still needed to connect to it). We created an Azure VNet and configured site-to-site connectivity that created a secure connection between the Azure VNet and their on-premises network.

We then had to configure point-to-site connections for the VNet [so that we could put the Azure Web App onto the VNet](https://azure.microsoft.com/en-us/documentation/articles/web-sites-integrate-with-vnet/). This would (in theory) allow the website to access their on-premises resources such as the database. We also had to upgrade the site to Standard pricing in order to do this.

We had to reconfigure the site-to-site gateway to allow dynamic routing in order to do this, which meant deleting and recreating the gateway. A bit of a pain, but not too bad. We then configured static routing from the on-premises network to the point-to-site addresses on the VNet.

## Ping from Azure Web App?

Once we had that all configured, we wanted to test connectivity. If we had deployed a VM, it would have been simple – just open a cmd prompt and ping away. However, we didn’t have a server, since we were deploying an Azure Web App. So initially we deployed a dummy Azure Web App onto the VNet to test the connection. This became a little bit of a pain. However, I remembered reading about Kudu and decided to see if that would be easier.

## Kudu to the Rescue

If you browse to [http://\<yoursite\>.scm.azurewebsites.net](http://<yoursite>.scm.azurewebsites.net) (where \<yoursite\> is the name of your Azure Web App) then you’ll see the Kudu site.

<!--kg-card-begin: html-->[![image](/assets/images/files/9d0dc450-7511-4540-901c-a39b7bf74d4b.png "image")](/assets/images/files/282b101d-ea87-4453-a22b-8e95a23c236d.png)<!--kg-card-end: html-->

Once you’ve opened the Kudu site, you can do all sorts of interesting things (see [this blog post](http://blogs.msdn.com/b/benjaminperkins/archive/2014/03/24/using-kudu-with-windows-azure-web-sites.aspx) and this [Scott Hanselman and David Ebbo video](https://channel9.msdn.com/Shows/Azure-Friday/What-is-Kudu-Azure-Web-Sites-Deployment-with-David-Ebbo)). If you open the Debug console (you can go CMD or PowerShell) then you get to play! I opened the CMD console and typed “help” – to my surprise I got a list of commands I could run:

<!--kg-card-begin: html-->[![image](/assets/images/files/bd5c54ff-2e65-4319-94a0-99d7381a4a6f.png "image")](/assets/images/files/01756426-fddc-496c-92c3-1e9924ede690.png)<!--kg-card-end: html-->

Unfortunately I didn’t see anything that would help me with testing connectivity. However, I remembered that I had read _somewhere_ about the command “tpcping”. So I tried it:

<!--kg-card-begin: html--><font size="3" face="Courier New">tcpping &lt;enter&gt;</font><!--kg-card-end: html--><!--kg-card-begin: html-->[![image](/assets/images/files/cf213b2f-ce32-45e9-ad58-9784cc048915.png "image")](/assets/images/files/9c7294f7-0294-40f4-832f-bc91f69dd34f.png)<!--kg-card-end: html-->

Looks promising! Even better than the “ping” command, you can also test for a specific port, not just the ip address. So I want to test if my site can reach my database server on port 1443, no problem:

<!--kg-card-begin: html--><font size="3" face="Courier New">tcpping 192.168.0.1:1443 &lt;enter&gt;</font><!--kg-card-end: html--><!--kg-card-begin: html-->[![image](/assets/images/files/e613500b-5d37-4a2c-90d5-6884dd4ff870.png "image")](/assets/images/files/00312b65-b46a-49ce-9414-b9411175fa4e.png)<!--kg-card-end: html-->

Hmm, seems that address isn’t working.

After troubleshooting for a while, we managed to sort the problem and tcpping gave us a nice “Success” message, so we knew we were good to go. Kudu saved us a lot of time!

Happy troubleshooting!

