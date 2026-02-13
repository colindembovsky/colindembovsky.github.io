---
layout: post
title: Application Insights Telemetry for WAWS or Customer-Hosted Sites Without MMA
date: '2014-04-18 02:25:00'
tags:
- appinsights
---

There are a couple of tools that change the development landscape significantly for .Net developers. One such tool is [IntelliTrace](http://msdn.microsoft.com/en-us/library/dd264915.aspx) – this revolutionizes how you debug production issues. Another game-changing tool is [Application Insights](http://msdn.microsoft.com/en-us/library/dn481095.aspx).

There are a few broad categories for Application Insights telemetry – APM (Application Performance Monitoring), Availability, Usage and Diagnostics. In order to leverage APM and Diagnostics, you need to install [Microsoft Monitoring Agent](http://go.microsoft.com/fwlink/?LinkID=328054) (MMA) on your IIS server. This isn’t a problem if you’re using an on-premises IIS or using your own VM in the cloud. However, if you’ve got a website that’s running on Windows Azure Websites (WAWS) or your site is going to be hosted on someone else’s tin or cloud, then you can’t install MMA. However, not all is lost – you can still get Availability and Usage telemetry for these scenarios.

## Add Application Insights to Your Web App

Normally this is as easy as installing the [Application Insights VS Extensions](http://go.microsoft.com/fwlink/?LinkID=390446), right-clicking your Web Application and selecting “Add Application Insights Telemetry”. This adds an ApplicationInsights.config file to your project that allows you to configure Application Insights. This works great in conjunction with MMA. Of course, the scenario we’re talking about is one where there is no MMA.

This scenario is supported, but there’s not a lot of guidance on how to do it. In cases where I’m running my site on WAWS or I’ve created a website that my customer is going to be hosting on their infrastructure (be that on their own premises, their own public cloud or even on Azure) and there’s no MMA, you can still get some really good telemetry. I also had the goal of minimizing admin and configuration footprint as much as possible, since I don’t want to have to maintain lots of config files on other people’s infrastructure. Happily, you can get a lot of good telemetry with a single application setting. Let’s see how to do it.

## Create an Application in Application Insights

For the WAWS (or customer-hosted website) scenarios, you need to create an application in Application Insights. This will generate a unique (guid) key for the application. This is the only piece of config you really need.

The first thing you need to do is “Choose Your Adventure” in Application Insights. There isn’t an adventure path for the scenario we’re trying, so we’ll hijack the Web Service adventure. Log into your VSO account and go to Application Insights. Click on “Overview” and then “Add Application”. Follow the prompts along the following path:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-aIBjS7Ia0yg/U1AOJL2dO2I/AAAAAAAABSk/ACMq-q7q8X4/image_thumb%25255B2%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-XVbm6nZskWo/U1AOIPHhUII/AAAAAAAABSc/wy57Wm7KnOM/s1600-h/image%25255B6%25255D.png)<!--kg-card-end: html-->

AppInsights will give us a list of instructions at this point. Since we’re going off-track on this adventure, we’ll ignore most of the instructions – we really just need the Application ID.

Type a name for your application into “Step 2” and click “Create”. Then copy the guid from Step 4 (this is the application ID we need to tie AppInsights events to this application in AppInsights):

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-MR1ORIZk2rE/U1AOLGNlXTI/AAAAAAAABS0/lvGC3oYZS84/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-w4UWdReOkZk/U1AOKe9R2vI/AAAAAAAABSs/bzZ3E93vDGc/s1600-h/image%25255B9%25255D.png)<!--kg-card-end: html-->

Note: You can get the application ID at any time by clicking on the gear icon in the top right of AppInsights. Then click the “Keys & Downloads” tab, select your application in the drop-down on the top left and scroll down to the “Web Services SDK” section. The guid there is the one you need for your application:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-fheSpW7Jre8/U1AONCgIVpI/AAAAAAAABTE/MXxX3fIjgWY/image_thumb%25255B16%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-GwSIoIRx7GY/U1AOMG6FcKI/AAAAAAAABS8/7h-Kx6t_j6o/s1600-h/image%25255B42%25255D.png)<!--kg-card-end: html-->

Follow the instructions in Step 3 – you can follow them here if you like. Select your web application in Solution Explorer, right-click and select “Manage NuGet packages”. Then type in “application insights” (including the quotes) and install the Application Insights SDK package.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-fchvYpOj7ko/U1AOQyAMFMI/AAAAAAAABTU/hofZBkiwmCk/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-OVHgGzNylOI/U1AOPXtkBHI/AAAAAAAABTM/tcvj2EdrEhI/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

Once that’s done, you can add the following code to your Global.asax.cs file:

    var appInsightsKey = ConfigurationManager.AppSettings["AppInsightsID"];
    if (string.IsNullOrEmpty(appInsightsKey))
    {
        ServerAnalytics.Enabled = false;
    }
    else
    {
        ServerAnalytics.Start(appInsightsKey);
    }

Now add a key to your application settings in your Web.config:

    <add key="AppInsightsID" value="guid">
    </add>

The value for the key is the guid you copied earlier when you created the Application in AppInsights. If you deploy this to another customer, create a new Application in AppInsights and update the key in the config file. That’s it.

## Setup Usage Telemetry from the Browser

To get usage telemetry from the browser (like Language, OS, Browser Type, Geographic Region etc.) you need to add some javascript to the pages you want to track. Back in Application Insights, you can click on the “Usage” tab. Make sure your new application is selected on the top left. Then click “Web site” for the type of application you want usage stats from. Click on “Click here to show instructions” at the bottom.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-dyGP963H2sI/U1AOTNEMIqI/AAAAAAAABTk/aFZVwGBRSu8/image_thumb%25255B4%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-Mdt2hsY-bZI/U1AOSArCA2I/AAAAAAAABTc/BD3qjbJj628/s1600-h/image%25255B12%25255D.png)<!--kg-card-end: html-->

If your site is on WAWS, you’ll want to create a ping test when you’ve published your application – if it’s already in Azure, then go ahead and enter the site url for the ping-test. This is a synthetic monitor that can give you a bit of basic availability information about your site. Of course when it’s on your customer’s infrastructure there may not even be a publically accessible url.

What’s important for now though is that you’ll see the usage insights snippet in step 3. You can copy that code if you like – though I made a slight modification to this code to make the application ID a parameter, rather than a hard-coded value.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-6Zns-J-36kY/U1AOVp-rX-I/AAAAAAAABT0/kb2mZVuf9Cs/image_thumb%25255B5%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-qyEtFamopx8/U1AOUdjW5UI/AAAAAAAABTs/ZFVzW0zl66E/s1600-h/image%25255B15%25255D.png)<!--kg-card-end: html-->

I’ve got an MVC application, so I’ve added the usage snippet javascript (mixed with a little Razor) into the \_Layout.cshtml page:

    <script type="text/javascript">
        var appInsightsKey = "@System.Configuration.ConfigurationManager.AppSettings["AppInsightsID"]";
        if (appInsightsKey !== null || appInsightsKey.length > 0) {
            window.appInsights = { queue: [], applicationInsightsId: null, accountId: null, appUserId: null, configUrl: null, start: function (n) { function u(n, t) { n[t] = function () { var i = arguments; n.queue.push(function () { n[t].apply(n, i) }) } } function f(n) { var t = document.createElement("script"); return t.type = "text/javascript", t.src = n, t.async = !0, t } function r() { i.appendChild(f("//az416426.vo.msecnd.net/scripts/ai.0.js")) } var i, t; this.applicationInsightsId = n; u(this, "logEvent"); u(this, "logPageView"); i = document.getElementsByTagName("script")[0].parentNode; this.configUrl === null ? r() : (t = f(this.configUrl), t.onload = r, t.onerror = r, i.appendChild(t)); this.start = function () { } } };
            appInsights.start(appInsightsKey);
            appInsights.logPageView();
        }
    </script>

You can see how I added a C# call (using Razor) to read the applicationID from the web.config – it’s the same call that the code in Global.asax.cs makes.

## Custom Events

So that will “buy” me some usage telemetry as well as page hits – all for free (almost). However, I wanted to add some specific events and even do some timing on events so that I can monitor performance. To do that I had to add some code into my methods.

You can log an event (just give it a name – the name allows a hierarchy separated by “/”). You can log a timed event – call StartTimedEvent() with the same naming convention. Later, you call event.End() or event.Cancel() to stop timing. Furthermore, you can log these events with metrics (and/or properties) using a Dictionary. Here are some snippets of stuff you can do:

Create a Timed Event for a Controller method:

    var aiEvent = ServerAnalytics.CurrentRequest.StartTimedEvent("DummySite/Index");
    
    try
    {
        // do stuff
        aiEvent.End();
    }
    catch(Exception ex)
    {
        // log stuff
        aiEvent.Cancel();
        throw;
    }
    
    return View();

Here’s just logging an event:

    public ActionResult Contact()
    {
        ServerAnalytics.CurrentRequest.LogEvent("DummySite/Contact");
        ViewBag.Message = "Your contact page.";
    
        return View();
    }

Here’s an event with some properties:

    public ActionResult About()
    {
        var even = DateTime.Now.Second % 2 == 0 ? "even" : "odd";
        var props = new Dictionary<string, object="">() { { "mod", even } };
    
        ServerAnalytics.CurrentRequest.LogEvent("DummySite/About", props);
    
        ViewBag.Message = "Your application description page.";
    
        return View();
    }</string,>

Of course you can add whatever custom events you need – the beauty of the API is that once you’ve called the Start method (in the Global.asax.cs) you don’t need to worry about IDs or config at all.

Now just publish and start monitoring! (Don’t forget to set your ping-test in the Availability tab in AppInsights if your site is publically available and you haven’t done it before).

## Viewing the Data in Application Insights

When you debug your application locally, you can check you events in the “Diagnostics-\>Streaming Data” page on AppInsights.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-3rPRTtsgn0U/U1AOX3U__iI/AAAAAAAABUE/hUIZgM-Comw/image_thumb%25255B7%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/--ElBYbnTezE/U1AOWwwdJcI/AAAAAAAABT8/7LhPcolip1A/s1600-h/image%25255B19%25255D.png)<!--kg-card-end: html-->

Once you’ve published your application, you’ll see events start flooding in.

In the Usage-\>Features-\>Events page, you’ll see your events, organized into the hierarchy you specified with the / notation:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-Fej5MvT6GvU/U1AOZVLlFQI/AAAAAAAABUU/6pxG6ZPuU34/image_thumb%25255B8%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-wHk0rCmjqHs/U1AOYuM5rbI/AAAAAAAABUM/ir1GBgw7TPg/s1600-h/image%25255B22%25255D.png)<!--kg-card-end: html-->

When I first saw this page, I couldn’t figure out where the timing for the timed events was. You can click on the little arrow (left of the pin icon for each event) or click the “DETAILS” link on the top left of the graph.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-v0ZpNg4z0EA/U1AObMADMBI/AAAAAAAABUk/N_yJE2bnA7s/image_thumb%25255B9%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-Sai19kQWT1U/U1AOafkyxYI/AAAAAAAABUc/rldKOij0GHo/s1600-h/image%25255B25%25255D.png)<!--kg-card-end: html-->

Clicking on the “dummysite/index” event arrow takes me to the details for that event, where I can analyze the timings:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-1uDtR4lTWgY/U1AOdT-jOlI/AAAAAAAABU0/KGI4stGfIjo/image_thumb%25255B10%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-qW1F_7vzKCQ/U1AOcFjxRsI/AAAAAAAABUs/ctnqln1btj8/s1600-h/image%25255B28%25255D.png)<!--kg-card-end: html-->

Clicking on the “dummysite/about” event, I get to see the properties (in this case, “mod” is the property and the values are “even” or “odd” – you can filter the graph by a particular value):

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-FTnQire5GtM/U1AOfb2FeLI/AAAAAAAABVE/aM3Z2JYjI-8/image_thumb%25255B11%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-uWAa1p7sGYo/U1AOeOU6oSI/AAAAAAAABU8/JEYDJlvHH2I/s1600-h/image%25255B31%25255D.png)<!--kg-card-end: html-->

The Usage-\>User page shows user/session specific telemetry – mine aren’t all that interesting since I only logged on with 1 user.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-2itUe45dl10/U1AOhMBh1wI/AAAAAAAABVU/69Bv9byGEdE/image_thumb%25255B13%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-iRFu0K1qULU/U1AOgJi5gKI/AAAAAAAABVM/a-dkyp7JggI/s1600-h/image%25255B35%25255D.png)<!--kg-card-end: html-->

Clicking on Usage-\>Environment gives me some telemetry around browsers, OSs, locations and so on.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-iOCFtrpmw9g/U1AOjfSVvwI/AAAAAAAABVg/ksDaNXF_2AA/image_thumb%25255B14%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-Ek3V2JJnHvg/U1AOiXayJBI/AAAAAAAABVc/109phYYQuzA/s1600-h/image%25255B38%25255D.png)<!--kg-card-end: html-->

All in all, it’s a LOT of data for only a couple of minor edits to code. You can also add \*some\* of this data to Dashboards, so you can make custom Dashboards for monitoring your sites.

Best of all, if I give this application to many different customers, all I need to do is supply them with a new AppInsightsID for their web.config file and I’ll instantly get telemetry. Very cool!

Happy monitoring!

