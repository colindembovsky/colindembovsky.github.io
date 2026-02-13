---
layout: post
title: Load Balancing DotNet Core Docker Containers with nginx
date: '2016-08-06 14:15:13'
tags:
- docker
---

Yes, I’ve been playing with Docker again – no big surprise there. This time I decided to take a look at scaling an application that’s in a Docker container. Scaling and load balancing are concepts you have to get your head around in a microservices architecture!

Another consideration when load balancing is of course shared memory. [Redis](http://redis.io/) is a popular mechanism for that (and since we’re talking Docker I should mention that there’s a [Docker image for Redis](https://hub.docker.com/_/redis/)) – but for this POC I decided to keep the code very simple so that I could see what happens on the networking layer. So I created a very simple .NET Core ASP.NET Web API project and added a single MVC page that could show me the name of the host machine. I then looked at a couple of load balancing options and started hacking until I could successfully (and easily) load balance three Docker container instances of the service.

## The Code

The code is stupid simple – for this POC I’m interested in configuring the load balancer more than anything, so that’s ok. Here’s the controller that we’ll be hitting:

    namespace NginXService.Controllers
    {
        public class HomeController : Controller
        {
            // GET: /&lt;controller&gt;/
            public IActionResult Index()
            {
                // platform agnostic call
                ViewData["Hostname"] = Environment.GetEnvironmentVariable("COMPUTERNAME") ??
                    Environment.GetEnvironmentVariable("HOSTNAME");
    
                return View();
            }
        }
    }

Getting the hostname is a bit tricky for a cross-platform app, since \*nix systems and windows use different environment variables to store the hostname. Hence the ?? code.

Here’s the View:

    @{
        &lt;h1&gt;Hello World!&lt;/h1&gt;
        &lt;br/&gt;
    
        &lt;h3&gt;Info&lt;/h3&gt;
        &lt;p&gt;&lt;b&gt;HostName:&lt;/b&gt; @ViewData["Hostname"]&lt;/p&gt;
        &lt;p&gt;&lt;b&gt;Time:&lt;/b&gt; @string.Format("{0:yyyy-MM-dd HH:mm:ss}", DateTime.Now)&lt;/p&gt;
    }

I had to change the Startup file to add the MVC route. I just changed the

<!--kg-card-begin: html--><font face="Courier New">app.UseMvc()</font><!--kg-card-end: html-->

line in the

<!--kg-card-begin: html--><font face="Courier New">Configure()</font><!--kg-card-end: html-->

method to this:

    app.UseMvc(routes =&gt;
    {
        routes.MapRoute(
            name: "default",
            template: "{controller=Home}/{action=Index}/{id?}");
    });

Finally, here’s the Dockerfile for the container that will be hosting the site:

    FROM microsoft/dotnet:1.0.0-core
    
    # Set the Working Directory
    WORKDIR /app
    
    # Configure the listening port
    ARG APP_PORT=5000
    ENV ASPNETCORE_URLS http://*:$APP_PORT
    EXPOSE $APP_PORT
    
    # Copy the app
    COPY . /app
    
    # Start the app
    ENTRYPOINT dotnet NginXService.dll
    

Pretty simple so far.

## Proxy Wars: HAProxy vs nginx

After doing some research it seemed to me that the serious contenders for load balancing Docker containers boiled down to [HAProxy](http://www.haproxy.org/) and [nginx](https://www.nginx.com/) (with corresponding Docker images [here](https://hub.docker.com/_/haproxy/) and [here](https://hub.docker.com/_/nginx/)). In the end I decided to go with nginx for two reasons: firstly, nginx can be used as a reverse proxy, but it can also serve static content, while HAProxy is just a proxy. Secondly, the nginx website is a lot cooler – seemed to me that nginx was more modern than HAProxy (#justsaying). There’s probably as much religious debate about which is better as there is about git rebase vs git merge. Anyway, I picked nginx.

## Configuring nginx

I quickly pulled the image for nginx (

<!--kg-card-begin: html--><font face="Courier New">docker pull nginx</font><!--kg-card-end: html-->

) and then set about figuring out how to configure it to load balance three other containers. I used a Docker volume to keep the config outside the container – that way I could tweak the config without having to rebuild the image. Also, since I was hoping to spin up numerous containers, I turned to [docker-compose](https://docs.docker.com/compose/). Let’s first look at the nginx configuration:

    worker_processes 1;
    
    events { worker_connections 1024; }
    
    http {
    
        sendfile on;
    
        # List of application servers
        upstream app_servers {
    
            server app1:5000;
            server app2:5000;
            server app3:5000;
    
        }
    
        # Configuration for the server
        server {
    
            # Running port
            listen [::]:5100;
            listen 5100;
    
            # Proxying the connections
            location / {
    
                proxy_pass http://app_servers;
                proxy_redirect off;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Host $server_name;
    
            }
        }
    }
    

This is really a bare-bones config for nginx. You can do a lot in the config. This config does a round-robin load balancing, but you can also configure least\_connected, provide weighting for each server and more. For the POC, there are a couple of important bits:

- Lines 10-16: this is the list of servers that nginx is going to be load balancing. I’ve used aliases (app1, app2 and app3, all on port 5000) which we’ll configure through docker-compose shortly.
- Lines 22-23: the nginx server itself will listen on port 5100.
- Line 26, 28: we’re passing all traffic on to the configured servers.

I’ve saved this config to a file called nginx.conf and put it into the same folder as the Dockerfile.

## Configuring the Cluster

To configure the whole cluster (nginx plus three instances of the app container) I use the following docker-compose yml file:

    version: '2'
    
    services:
      app1:
        image: colin/nginxservice:latest
      app2:
        image: colin/nginxservice:latest
      app3:
        image: colin/nginxservice:latest
    
      nginx:
        image: nginx
        links:
         - app1:app1
         - app2:app2
         - app3:app3
        ports:
         - "5100:5100"
        volumes:
         - ./nginx.conf:/etc/nginx/nginx.conf

That’s 20 lines of code to configure a cluster – pretty sweet! Let’s take a quick look at the file:

- Lines 4-9: Spin up three containers using the image containing the app (that I built separately, since I couldn’t figure out how to build and use the same image multiple times in a docker-compose file).
- Line 12: Spin up a container based on the stock nginx image.
- Lines 13-16: Here’s the interesting bit: we tell docker to create links between the nginx container and the other containers, aliasing them with the same names. Docker creates internal networking (so it’s not exposed publically) between the containers. This is very cool – the nginx container can reference app1, app2 and app3 (as we did in the nginx config file) and docker takes care of figuring out the IP addresses on the internal network.
- Line 18: map port 5100 on the nginx container to an exposed port 5100 on the host (remember we configured nginx to listen on the internal 5100 port).
- Line 20: map the nginx.conf file on the host to /etc/nginx/nginx.conf within the container.

Now we can simply run

<!--kg-card-begin: html--><font face="Courier New">docker-compose up</font><!--kg-card-end: html-->

to run the cluster!

<!--kg-card-begin: html--> [![image](/assets/images/files/70a176b1-99a8-45d5-8946-ead54c69b254.png "image")](/assets/images/files/dc796082-cfca-4a0a-95ed-956160f52b5b.png)<!--kg-card-end: html-->

You can see how docker-compose pulls the logs into a single stream and even color-codes them!

The one thing I couldn’t figure out was how to do a

<!--kg-card-begin: html--><font face="Courier New">docker build</font><!--kg-card-end: html-->

on an image and use that image in another container within the docker-compose file. I could just have three

<!--kg-card-begin: html--><font face="Courier New">build</font><!--kg-card-end: html-->

directives, but that felt a bit strange to me since I wanted to supply build args for the image. So I ended up doing the

<!--kg-card-begin: html--><font face="Courier New">docker build</font><!--kg-card-end: html-->

to create the app image and then just using the image in the docker-compose file.

Let’s hit the index page and then refresh a couple times:

<!--kg-card-begin: html-->[![image](/assets/images/files/7b82771f-1ffd-474d-9d28-b4e9f3a6fef3.png "image")](/assets/images/files/b9385ace-8038-4e6c-88e1-de26aa9a939a.png)<!--kg-card-end: html--><!--kg-card-begin: html-->[![image](/assets/images/files/c8494cea-7c5b-4589-b7f9-b80981e095f8.png "image")](/assets/images/files/2566950d-3709-4fcf-a654-41fceda99e2e.png)<!--kg-card-end: html--><!--kg-card-begin: html-->[![image](/assets/images/files/d2128d96-b76c-4634-9cbc-4447fb7b749b.png "image")](/assets/images/files/d55d4238-0095-4c19-b213-70c60b4a1a12.png)<!--kg-card-end: html-->

You can see in the site (the hostname) as well as in the logs how the containers are round-robining:

<!--kg-card-begin: html-->[![image](/assets/images/files/454927e0-2492-4a6f-a21f-fce3580dac99.png "image")](/assets/images/files/785447ca-9765-44d4-a299-f7d1932ecb87.png)<!--kg-card-end: html-->
## Conclusion

Load balancing containers with nginx is fairly easy to accomplish. Of course the app servers don’t need to be running .NET apps – nginx doesn’t really care, since it’s just directing traffic. However, I was pleased that I could get this working so painlessly.

Happy load balancing!

