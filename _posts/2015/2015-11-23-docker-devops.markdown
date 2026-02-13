---
layout: post
title: Docker DevOps
date: '2015-11-23 16:02:06'
tags:
- devops
- cloud
---

Recently I attended the MVP Summit in Redmond. This is an annual event where MVPs from around the world converge on Microsoft to meet with each other and various product teams. It’s a highlight of the year (and one of the best benefits of being an MVP).

<!--kg-card-begin: html-->[![image](/assets/images/files/0fead412-36f9-4855-b0da-db677c5c1b58.png "image")](/assets/images/files/bf5182e4-d1ac-4e2b-9ae5-0bfcc00f631c.png)<!--kg-card-end: html-->

The ALM MVPs have a tradition – we love to hear what other MVPs have been doing, so we have a pre-Summit session where we get 20 minute slots to share anything that’s interesting. This year I did a slide ware chat entitle _Docker DevOps_. It was just a collection of thoughts that I have on what Docker means for DevOps. I’d like to put some of those thoughts down in this post.

## Docker Means Containers

[Docker](https://docs.docker.com/) isn’t actually a technology per se. It’s just a containerization manager that happened to be at the right place at the right time – it’s made containers famous.

Container technology has been around for a fairly long time – most notably in the Linux kernel. Think of containers as the evolution of virtualization. When you have a physical server, it can be idle a lot of the time. So virtualization became popular, allowing us to create several virtual machines (VMs) on a single server. Apps running on the VM don’t know they’re on a VM – the VM has abstracted the physical hardware. Now most developers and IT Pros take virtualization for granted.

Containers take the abstraction deeper – they abstract the OS too. Containers are running instances of _images._ The base layer of the image is typically a lightweight OS – only the bare essentials needed to run an app. Typically that means no UI or anything else that isn’t strictly needed. Images are also immutable. Under the hood, when you change an image, you actually create a differencing layer on top of the current layer. Containers also share layers – for example, if two containers have an ubuntu14.04 base layer, and then one has nginx and another has mySQL, there’s only one physical copy of the ubuntu14.04 image on disk. Shipping containers means just shipping the different top layer, which makes them easily portable.

### Windows Containers

So what about [Windows containers](https://msdn.microsoft.com/virtualization/windowscontainers/containers_welcome)? Windows Server 2016 TP 4 (the latest release at the time of this article) has support for Windows containers – the first OS from Microsoft to support containerization. There are two flavors of Windows container – Windows Server containers and Hyper-V containers. Windows Server container processes are visible on the host, while Hyper-V containers are completely “black box” as far as the host is concerned – that makes the HyperV containers “more secure” than Windows Server containers. You can switch the mode at any time.

Windows container technology is still in its infancy, so there are a few rough edges, but it does show that Microsoft is investing in container technology. Another glaring sign is the fact that you can already create Docker hosts in Azure (both for Windows and Linux containers). Microsoft is also actively working on open-source Docker.

## What Containers Mean For You

So what does it all mean for you? Here’s the rub – just like you’ve probably not installed a _physical_ server for some years because of virtualization, I predict that pretty soon you won’t even install and manage VMs anymore. You’ll have a “cloud of hosts” somewhere (you won’t care where) and have the ability to spin up containers to your heart’s content. In short, it’s the way of the future.

So here are some things you need to be thinking about if you want to ride the wave of the future:

- Microservices
- Infrastructure as Code
- Immutable machines
- Orchestration
- Docker Repositories
- It works on my machine

### Microservices

The most important architectural change that containers bring is _microservices_. In order to use containers effectively, you have to (re-)architect your applications into small, loosely coupled services (each deployed into its own container). This makes each individual service simpler, but moves quite a bit of complexity into managing the services. Coordinating all these microservices is a challenge. However, I believe that the complexity at the management level is – well, more manageable. If done correctly, microservices can be deployed without much (or any) impact to other services, so you can isolate issues, deploy smaller units more frequently and gain scalability in the parts of the overall application that require it, as and when they require it (this is the job of the orchestration engine – something I’ll talk to later). This is much better than having to deploy an entire monolithic application every time.

So what about networking between the containers? Turns out that Docker is pretty good at managing how containers talk to each other (via [Docker Swarm](https://docs.docker.com/swarm/) and [Docker Compose](https://docs.docker.com/compose/)). Each container must define which ports it exposes (if any). You can also link containers, so that they can communicate with each other. Furthermore, you have to explicitly define a “mapping” between the container ports and the host ports in order for the container to be exposed outside its host machine. So you have tight control over the surface area of each container (or group of containers). But it’s another thing to manage.

### Infrastructure as Code

When you create a Docker image, you specify it in a Dockerfile. The Dockerfile contains instructions in text that tell the Docker engine how to build up an image. The starting layer typically the (minimal) OS. Then follow instructions to install dependencies that the top app layers will need. Finally, the app itself is added.

Specifying your containers in this manner forces you to express your _infrastructure_ as code. This is a great practice, whether you’re doing it for Docker or not. After you’ve described your infrastructure as code, you can automate building the infrastructure – so Infrastructure as Code is a building block for automation. Automation is good – it allows rapid and reliable deployment, which means better quality, faster. It does mean that you’re going to have to embrace DevOps – and have both developers and operations (or better yet your DevOps team) work together to define and manage the infrastructure code. In this brave new world, no-one is installing OSs or anything else using GUIs. Script it baby, script it!

### Immutable Machines

Containers are essentially _immutable_. Under the hood, if you change a container, you actually freeze the current top layer (so that it’s immutable) and add a new “top layer” with the changes (this is enabled by [Union File Systems](https://docs.docker.com/engine/reference/glossary/#union-file-system)). In fact, if you do it correctly, you should never have a reason to change a container once it’s out of development. If you really do need to change something (or say, deploy some new code for your app within the container), you actually throw away the existing container and create a new one. Don’t worry though – Docker is super efficient – which means that you won’t need to rebuild the entire image from scratch – the interim layers are stored in the Docker engine, so Docker is smart enough to just use the common layers again and just create a new differencing layer for the new image.

Be that as it may, there is a shift in thinking about containers in production. They should essentially be viewed as immutable. Which means that your containers have to be _stateless_. That obviously won’t work for databases or any other persistent data. So Docker has the concept of _[data volumes](https://docs.docker.com/engine/userguide/dockervolumes/)_, which are special directories that can be accessed (and shared) by containers but that are outside the containers themselves. This means you have to really think about where the data are located for containers and where they live _on the host_ (since they’re outside the containers). Migrating or upgrading data is a bit tricky with so many moving parts, so it’s something to think about carefully.

### Orchestration

So let’s imagine that you’ve architected an application composed of several microservices that can be deployed independently. You can spin them all up on a single machine and then – hang on, a single machine? Won’t that hit resource limitations pretty quickly? Yes it will. And what about the promise of scale – that if a container comes under pressure I can just spin another instance (container) up and voila – I’ve scaled out? Won’t that depend on how much host resources are available? Right again.

This is where tools like Docker Swarm come into play. Docker Swarm allows you to create and access a pool of Docker hosts. Ok, that’s great for deploying apps. But what about monitoring the resources available? And wouldn’t it be nice if the system could auto-scale? Enter [Apache Mesos](http://mesos.apache.org/) and [Mesosphere](https://mesosphere.com/) (there are other products in this space too). Think of Mesos as a _distributed kernel_. It aggregates a bunch of machines – be they physical, virtual or cloud – into what appears to be a single machine that you can program against. Mesosphere is then a layer on top of Mesos that further abstracts, allowing much easier consumption and use of the Datacenter OS (dcos), which enables highly available, highly automated systems. Mesos uses containers natively, so Docker works in Mesos and Mesosphere. If you’re going to build scalable apps, then you are going to need an orchestration engine like Mesosphere. And it [runs in Azure](https://mesosphere.com/blog/2015/09/29/mesosphere-and-mesos-power-the-microsoft-azure-container-service/) too!

### Docker Repositories

Docker enables you to define a container (or image) using a Dockerfile. This file can be shared via some code repository. Then developers can code against that container (by building it locally) and when it’s ready for production, Ops can pull the file down and build the container. Sounds like a great way to share and automate! Docker repositories allow you to share Dockerfiles in exactly this manner. There are public repos, like [DockerHub](https://hub.docker.com/), and you can of course create (or subscribe) to private repos. This means that you get to share base images from official partners (for example, if you need nginx, no need to build it yourself – just pull down the official image from DockerHub that the nginx guys themselves have built). It also means that you have a great mechanism for moving code from dev to QA to Production. Just share the images in a public (or private) repo, and if a tester wants to test they can just spin up a container or two for themselves. The containers run exactly the same wherever they are run, so it could be the developer’s laptop, in a Dev/Test lab or in Production. And since only the delta’s are actually moved around (common images are shared) it’s quick and efficient to share code in this manner.

### It Works on My Machine
<figure class="kg-card kg-image-card"><img src="http://wpup.codeimpossible.com/2009/06/works-on-my-machine-starburst.jpg" class="kg-image" alt loading="lazy"></figure>

“It works on my machine!” The classic exclamation heard by developer’s world over every time a bug is filed. And we all laugh since we know that between your dev environment and Production lie a whole slew of differences. Except that now, since the containers run the same wherever they are run, if it works in the developer’s container, it works in the Prod container.

Of course there are ways the containers may differ – for example, most real-world containers will have environment variables that have different values in different environments. But containers actually allow “It works on my machine” to become a viable statement once more.

## Conclusion

Containers are the way of the future. You want to make sure that you’re getting on top of containers early (as in now) so that you don’t get left behind. Start re-architecting your application into microservices, and start investigating hosting options, starting with Docker and Docker Compose and moving towards dcos like Mesosphere. And be proud, once more, to say, “It works on my machine!”

Happy containering!

