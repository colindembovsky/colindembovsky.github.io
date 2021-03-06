---
layout: post
title: 'Container DevOps: Beyond Build (Part 4) - Telemetry with Prometheus'
date: '2019-05-08 09:37:11'
tags:
- docker
---

1. TOC
{:toc}

Series:

- [Part 1: Intro](/container-devops-beyond-build-part-1)
- [Part 2: Traefik Basics](/container-devops-beyond-build-part-2---traefik)
- [Part 3: Canary Testing](/container-devops-beyond-build-part-3---canary-testing)
- Part 4: Telemetry with Prometheus (this post)
- [Part 5: Prometheus Operator](/container-devops-beyond-build-part-5---prometheus-operator)

In my [previous post](/container-devops-beyond-build-part-3---canary-testing) in this series I wrote about how I used [Traefik](https://traefik.io/) to do traffic shifting and canary testing. I asserted that without proper telemetry, canary testing is (almost) pointless. Without some way to determine the efficacy of a canary deployment, you may as well just deploy straight out and not pretend.

I've also written about how I love and use [Application Insights to monitor .NET applications](/appinsights-analytics-in-the-real-world). Application Insights (or AppInsights for short) is still my go-to telemetry tool. And it's not only a .NET tool - there are SDKs for Java, Javascript and Python among others. But since we're delving into container-land, I wanted to at least explore one of the popular k8s tools: [Prometheus](https://prometheus.io/). There are other monitoring tools (like [Datadog](https://www.datadoghq.com/)) and I think it'll be worth doing a compare/contrast of various monitoring tools at some stage. But for this post, I'll stick to Prometheus.

## Business vs Performance Telemetry

Most developers that are using any kind of telemetry understand "performance" telemetry - requests per second, read/writes per second, errors per second, memory and CPU usage - usual, bread-and-butter telemetry. However, I often encourage teams not to stop at _performance_ telemetry and to also start looking at how to instrument their applications with "business" telemetry. _Business telemetry_ is telemetry that has nothing to do with the running application - and everything to do with how the site or application is doing in business terms. For example, how many products of a certain category were sold today? What products are popular in which geos? And so on.

AppInsights is one of my go-to tools because you get performance telemetry "for free" - just add it to your project and you get all of the perf telemetry you need to have a good view of your application performance - and that's without changing a single line of code! However, if you do want business telemetry, you can add a few lines of code and it's simple to get business telemetry. Add to that the ability to connect PowerBI to your telemetry (something I've written about [before](/appinsights-analytics-in-the-real-world)) and you're able to produce the telemetry and have business users consume it using PowerBI - that's a recipe for success!

On the down-side, making sense of AppInsights telemetry definitely isn't simple, and the learning curve for analyzing your data is steep. The AppInsights query language is a delight though, and even has some built-in [machine learning capabilities](https://docs.microsoft.com/en-us/azure/azure-monitor/app/proactive-diagnostics)).

## Prometheus

Prometheus has long been a popular telemetry solution - however, as I was exploring it I came across some challenges. Firstly, integrating into .NET isn't simple - and you don't get anything "for free" - you have to code in the telemetry. Secondly, there are only four types of metrics you can [utilize](https://prometheus.io/docs/concepts/metric_types/): Counter, Gauge, Histogram and Summary. These are great for performance telemetry, but are very difficult to use for business telemetry. However, creating graphs from Prometheus data is really simple (at least using [Grafana](https://grafana.com/), as I'll discuss in a later post) and there's a whole query language called [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/) for querying Prometheus metrics.

In the remainder of this post I'll show how I used Prometheus in a .NET Core application.

## Performance Telemetry

To add performance telemetry to a .NET Core application, you have to add some middleware. You also need to expose the Prometheus endpoint. Here's a snippet from my Startup.cs file:

    using Prometheus;
    
    public class Startup
    {
        public void Configure(IApplicationBuilder app)
        {
            var basePath = Configuration["PathBase"] ?? "/";
            ...
    
            // prometheus
            var version = Assembly.GetEntryAssembly().GetCustomAttribute<assemblyfileversionattribute>()
                    .Version.ToString();
            app.UseMethodTracking(version, Configuration["ASPNETCORE_ENVIRONMENT"], Configuration["CANARY"]);
            app.UseMetricServer($"{basePath}/metrics");
    </assemblyfileversionattribute>

Notes:

- Line 1: Import the Prometheus namespace - this is from the [Prometheus NuGet package](https://www.nuget.org/packages/prometheus-net.AspNetCore/)
- Line 7: We need to set a base path - this is for sharing Traefik frontends for multiple backend services
- Lines 10-11: Get the version of the application
- Line 12: Call the UseMethodTracking method (shown below) to configure middleware, passing in the version, environment and canary name
- Line 13: Tell Prometheus to expose an endpoint for the Prometheus server to scrape

I want my metrics to be dimensioned by version, environment and canary. This is critical for successful canary testing! We also need a pathbase other than "/" since when we deploy services behind the Traefik router, we want to use path-based rules to route traffic to different backend services, even though there's only a single front-end base URL.

Here's the code for the UseMethodTracking method:

    public static class PrometheusAppExtensions
    {
        static readonly string[] labelNames = new[] { "version", "environment", "canary", "method", "statuscode", "controller", "action" };
    
        static readonly Counter counter = Metrics.CreateCounter("http_requests_received_total", "Counts requests to endpoints", new CounterConfiguration
        {
            LabelNames = labelNames
        });
    
        static readonly Gauge inProgressGauge = Metrics.CreateGauge("http_requests_in_progress", "Counts requests currently in progress", new GaugeConfiguration
        {
            LabelNames = labelNames
        });
    
        static readonly Histogram requestHisto = Metrics.CreateHistogram("http_request_duration_seconds", "Duration of requests to endpoints", new HistogramConfiguration
        {
            LabelNames = labelNames
        });
    
        public static void UseMethodTracking(this IApplicationBuilder app, string version, string environment, string canary)
        {
            app.Use(async (context, next) =&gt;
            {
                // extract values for this event
                var routeData = context.GetRouteData();
                var action = routeData?.Values["Action"] as string ?? "";
                var controller = routeData?.Values["Controller"] as string ?? "";
                var labels = new string[] { version, environment, canary,
                    context.Request.Method, context.Response.StatusCode.ToString(), controller, action };
    
                // start a timer for the histogram
                var stopWatch = Stopwatch.StartNew();
                using (inProgressGauge.WithLabels(labels).TrackInProgress()) // increments the inProgress, decrementing when disposed
                {
                    try
                    {
                        await next.Invoke();
                    }
                    finally
                    {
                        // record the duration
                        stopWatch.Stop();
                        requestHisto.WithLabels(labels).Observe(stopWatch.Elapsed.TotalSeconds);
    
                        // increment the counter
                        counter.WithLabels(labels).Inc();
                    }
                }
            });
        }
    }

Notes:

- Line 2: set up the names of the dimensions I want to configure - note version, environment and canary
- Lines 5-8: set up a counter to count method hits
- Lines 10-13: set up a gauge to report how many requests are in progress
- Lines 15-18: set up a histogram to record duration of each method call
- Line 20: create a static extension method to inject Prometheus tracking into the middleware
- Line 22: add a new handler into the pipeline
- Lines 25-28: extract action, controller, method and response code from the current request if available
- Line 32: start a stopwatch
- Line 33: tell Prometheus that a method call is in progress - the "end of operation" is automatic at the end of the using (line 47)
- Line 35-38: invoke the actual request
- Line 39-43: stop the stopwatch and log the time recorded
- Line 46: increment the counter for this controller/action/method/version/environment/canary combination

This code gives us performance metrics - we inject a step into the pipeline that starts a stopwatch, calls the operation, tells Prometheus an operation is in progress, and then when the operation completes, records the time taken and increments the call counter. Each "log" includes the "withLabels()" call that creates the context (dimensions) for the event.

## Total Sales by Product Category

Let's examine what telemetry looks like if we want to track a business metric: say, sales of products by category. For this to work, I'd need to track the product category and price of each item sold. I could add other dimensions too (such as user) so that I can extend my analytics. If I know which users are purchasing products, I can start slicing and dicing by geo or language or other user attributes. If I know when sales occur, I can slice and dice by day of week or hour or any other time-based dimensions. The more dimensions I have, the more insights I can drive.

Let's see how we would track this business metric using Prometheus. Firstly, which metric type do I need? If we use a Counter, we can count how many items are sold, but not track the price - because counters can only increment by 1, not anything else. I could try Gauge since Gauge lets me set an arbitrary number - but unfortunately that doesn't give me a running total - it's just a number at a point in time. Both Histogram and Summary are snapshots of observations in a time period (my wording) so they don't work either. In the end I decided to settle for number of products sold as a proxy for revenue - each time an item is sold, I want to log a counter for the product category and other dimensions so I get some idea of business telemetry.

Generally I like a logging framework with an interface that abstracts the logging mechanism or storage away from the application. I found that this was relatively easy to do using AppInsights - however, Prometheus doesn't really work that way because the metric types are very specific to a particular event or method.

Here's how I ended up logging some telemetry in Prometheus in my .NET Core application:

    public class ShoppingCartController : Controller
    {
        static readonly string[] labelNames = new[] { "category", "product", "version", "environment", "canary" };
    
        readonly Counter productCounter = Metrics.CreateCounter(
            "pu_product_add", "Increments when product is added to basket", 
            new CounterConfiguration
            {
                LabelNames = labelNames
            });
        
        public async Task<iactionresult> AddToCart(int id)
        {
            // Retrieve the product from the database
            var addedProduct = _db.Products
                .Include(product =&gt; product.Category)
                .Single(product =&gt; product.ProductId == id);
    
            var labels = new string[] { properties["ProductCategory"], properties["Product"], version, environment, canary };
            productCounter.WithLabels(labels).Inc();
            ...
        }
        ...
    }
    </iactionresult>

Notes:

- Line 3: Set up a list of label names - again, these are dimensions for the telemetry
- Lines 5-10: Set up a Counter for counting when a product is added to a basket, using labelNames for the dimensions
- Line 19: Create an array of values that correspond to the labelNames array
- Line 20: Increment the counter, again using WithLabels()

## Viewing Telemetry

Now that we have telemetry integrated, we can view the telemetry by browsing to the endpoint we configured Prometheus to expose. We'll get some of the metrics live:

    # HELP process_open_handles Number of open handles
    # TYPE process_open_handles gauge
    process_open_handles 346
    # HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
    # TYPE process_start_time_seconds gauge
    process_start_time_seconds 1556149570.76
    # HELP dotnet_total_memory_bytes Total known allocated memory
    # TYPE dotnet_total_memory_bytes gauge
    dotnet_total_memory_bytes 8133304
    # HELP process_virtual_memory_bytes Virtual memory size in bytes.
    # TYPE process_virtual_memory_bytes gauge
    process_virtual_memory_bytes 12298391552
    # HELP http_request_duration_seconds Duration of requests to endpoints
    # TYPE http_request_duration_seconds histogram
    http_request_duration_seconds_sum{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action=""} 6.6218794
    http_request_duration_seconds_count{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action=""} 926
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="0.005"} 872
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="0.01"} 909
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="0.025"} 916
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="0.05"} 919
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="0.075"} 919
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="0.1"} 922
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="0.25"} 923
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="0.5"} 925
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="0.75"} 925
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="1"} 925
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="2.5"} 925
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="5"} 926
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="7.5"} 926
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="10"} 926
    http_request_duration_seconds_bucket{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action="",le="+Inf"} 926
    # HELP process_num_threads Total number of threads
    # TYPE process_num_threads gauge
    process_num_threads 24
    # HELP dotnet_collection_count_total GC collection count
    # TYPE dotnet_collection_count_total counter
    dotnet_collection_count_total{generation="0"} 3
    dotnet_collection_count_total{generation="2"} 0
    dotnet_collection_count_total{generation="1"} 1
    # HELP process_working_set_bytes Process working set
    # TYPE process_working_set_bytes gauge
    process_working_set_bytes 159961088
    # HELP http_requests_received_total Counts requests to endpoints
    # TYPE http_requests_received_total counter
    http_requests_received_total{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action=""} 926
    # HELP process_private_memory_bytes Process private memory size
    # TYPE process_private_memory_bytes gauge
    process_private_memory_bytes 0
    # HELP pu_product_add Increments when product is added to basket
    # TYPE pu_product_add counter
    pu_product_add{category="Wheels &amp; Tires",product="Disk and Pad Combo",version="1.0.0.45",environment="Production",canary="blue"} 1
    pu_product_add{category="Oil",product="Oil and Filter Combo",version="1.0.0.45",environment="Production",canary="blue"} 1
    # HELP http_requests_in_progress Counts requests currently in progress
    # TYPE http_requests_in_progress gauge
    http_requests_in_progress{version="1.0.0.45",environment="Production",canary="blue",method="GET",statuscode="200",controller="",action=""} 1
    # HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
    # TYPE process_cpu_seconds_total counter
    process_cpu_seconds_total 15.26

Notes:

- Lines 14-32: shows http request duration in buckets - notice how each bucket also has the version of the app, the environment, canary, statuscode, controller and action
- Lines 50-53: shows the number of products added to the basket by category, product, version, environment and canary

## Conclusion

I didn't get round to showing how Prometheus scrapes the metrics from various services so that you can start to dashboard and analyze - that's the subject for the next post. While I find Prometheus is fairly painful to implement on the tracking side (certainly compared to AppInsights), the graphing and querying can be worth the pain. I'll show you how to do that in a k8s cluster in the next post.

Unfortunately, though I experimented with using Prometheus for "business" telemetry, I can't say I recommend it. It's really meant for performance telemetry. So use AppInsights - which you can totally do even from within containers - if you need to do any business telemetry. And you do!

Happy monitoring!

