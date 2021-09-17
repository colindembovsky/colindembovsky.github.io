---
layout: post
title: 'Container DevOps: Beyond Build (Part 1)'
date: '2019-05-08 09:36:17'
tags:
- docker
---

1. TOC
{:toc}

Series:

- Part 1: Intro (this post)
- [Part 2: Traefik Basics](/container-devops-beyond-build-part-2---traefik)
- [Part 3: Canary Testing](/container-devops-beyond-build-part-3---canary-testing)
- [Part 4: Telemetry with Prometheus](/container-devops-beyond-build-part-4---telemetry-with-prometheus)
- [Part 5: Prometheus Operator](/container-devops-beyond-build-part-5---prometheus-operator)

I've written before that I think that containers - and Kubernetes (k8s) - are the way of the future. I was fortunate enough to attend my first KubeCon last year in Seattle, and I was happy to see the uptake of k8s and the supporting cloud native technologies around k8s are robust and healthy. But navigating the myriad of services, libraries and techniques is a challenge! This is going to be the first in a series of posts about Container DevOps - and I don't just mean building images and deploying them. What about monitoring? And A/B testing? And all the other stuff that successful DevOps teams are supposed to be doing? We'll look at how you can implement some of these tools and techniques in this series.

## PartsUnlimited 1.0

For the last couple of years I've had the opportunity to demo DevOps using Azure DevOps probably a few hundred times. I built a demo in Azure DevOps using a fork of Microsoft's [PartsUnlimited repo](https://github.com/microsoft/partsunlimited). When I originally built the demo, the .NET Core tooling was a bit of a mess, so I just stuck with the full framework version. The demo targets Azure App Services and shows how you can make a change in code, then submit a Pull Request, which triggers a build that compiles, runs static code analysis and unit tests and triggers a release, which deploys the new version of the app to a staging slot in the Azure web app, routing some small percentage of all traffic to the slot for canary testing. All the while metrics are being collected in Application Insights, and after doing the canary deployment, you can analyze the metrics and decide if the canary is successful or not - and then either promote it to the rest of the site or just shift all traffic to the existing prod version in the case of a failure.

But how do you do the same sort of thing with k8s? That's what I set out to discover. But before we get there, let's take a step back and consider what we should be investigating in the world of Container DevOps in the first place!

## Components of Container DevOps

Here are some of the components of Container DevOps that I think need to be considered:

1. Building Quality - including multi-stage image builds, reportable unit testing and publishing to secure image repositories
2. Environment isolation - isolating Dev, Test and Prod environments
3. Canary Testing - testing changes on a small set or percentage of users before promoting to everyone (also called Testing in Production or Blue/Green testing)
4. Monitoring - producing, consuming and analyzing metrics about your services
5. Security - securing your services using TLS
6. Resiliency - making services resilient through throttling or circuit-breakers
7. Infrastructure and Configuration as Code

There are some more that I think should be on the list that I haven't yet gotten to exploring in detail - such as vulnerability scanning. Hopefully I get around to adding to the above list, but we'll start here for now.

### Building Quality

I've [previously blogged](/net-core-multi-stage-dockerfile-with-test-and-code-coverage-in-azure-pipelines) about how to run unit tests - and publish the test and code coverage results - in Azure DevOps pipelines. It was a good exercise, but as I look at it now I realize why I prefer to build code outside the container and copy the binaries in: it's hard to do ancillary work (like unit test, code scans etc.) in a Dockerfile. One advantage to the multi-stage Dockerfile that you'll lose is the dependency management - you have to manage that on the build machine (or container) if you're building the code outside the Dockerfile. But I think the dependency management ends up being simpler than trying to run (and publish) tests and test coverage and static analysis inside the Dockerfile. My post covered how to do unit testing/code coverage, but when I thought about adding SonarQube analysis or vulnerability scanning with WhiteSource, I realized the Dockerfile starts becoming clumsy. I think it's easier to just drop in the SonarQube and WhiteSource tasks into a pipeline and build on the build machine - and then just have a Dockerfile copy the compiled binaries in to create the final light-weight container image.

### Environment Isolation

There are a couple of ways to do this: the most isolated (and expensive) is to spin up a k8s cluster per environment - but you can achieve good isolation using k8s namespaces. You could also use a combination: have a Prod cluster and a Dev/Test cluster that uses namespaces to separate environments within that cluster.

### Canary Testing

This one took a while for me to wrap my head around. Initially I struggled with this concept because I was too married to the Azure Web App version, which works as follows:

1. [Traffic Manager](/testing-in-production-routing-traffic-during-a-release) routes 0% traffic to the "blue" slot
2. Deploy the new build to the blue slot - it doesn't matter if this slot is down, since no traffic is incoming anyway
3. Update Traffic Manager to divert some percentage of traffic to the blue slot
4. Monitor metrics
5. If successful, swap the "blue" slot with the production slot (an instantaneous action) and update Traffic Manager to route 100% of traffic to the new production slot

I started trying to achieve the same thing in k8s, but k8s doesn't have the notion of slots. An easy enough solution is to have a separate Deployment (and Service) for "blue" and "green" versions. But then there's no Traffic Manager - so I stated investigating various [Ingresses](https://kubernetes.io/docs/concepts/services-networking/ingress/) and Ingress Controllers to see if I could get the same sort of functionality. I initially got a POC running in [Istio](https://istio.io/) - but was still "swapping slots" in my mental model. Unfortunately, Istio, while very capable, is large and complicated - it felt like a sledge-hammer when all I needed was a screwdriver. I then tried [linkerd](https://linkerd.io/) - which was fine until I hit some limitations. Finally, I tried [Traefik](https://traefik.io/) - and I found I could do everything I wanted to (and more) using Traefik. There's definitely a follow on post here detailing this part of my Container DevOps journey - so stay tuned!

The other mental breakthrough came when I realized that Deployments (unlike slots in Azure Web Apps) are inherently highly available: that is, I can deploy a new version of a Deployment and k8s automatically does rolling updates to ensure the Deployment is never "down". So I didn't have to worry about diverting traffic away from the "blue" Deployment while I was busy updating it - and I can even keep the traffic split permanent. What that means is that I have two versions of the Deployment/Service: a "blue" one and a "green" one. I set up traffic splitting using Traefik and route say 20% of traffic to the "blue" service. When rolling out a new version, I simply update the tag of the image in the Deployment and k8s automatically does a rolling update for me - the blue slot never goes down and it's "suddenly" on the new version. Then I can monitor metrics, and if successful, I can update the "green" Deployment. Now both Blue and Green Deployments are on the new version, so while the Blue Deployment is getting 20% of the traffic, the versions are the same, so 100% of the traffic is now on the new version - and I had zero downtime! Of course I can simply revert the Blue Deployment back to the old version to get the same effect if the experiment is NOT a success.

One more snippet to whet your appetite for future posts on this topic: Traefik also handles TLS certs via LetsEncrypt, circuit-breaking and more.

### Monitoring

I am a huge fan of [Application Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) - and there's no reason not to continue logging to AppInsights from within your containers - assuming your containers can reach out to Azure. However, I wanted to see how I could do monitoring completely "in-cluster" and so I turned to [Prometheus](https://prometheus.io/) and [Grafana](https://grafana.com/). This is also a subject for another blog post, but I managed to (without too much hassle) get the following to work:

1. Export Prometheus metrics from .NET Core containers
2. Create a [Prometheus Operator](https://github.com/coreos/prometheus-operator) in my k8s cluster to easily (and declaratively) add Prometheus scraping to new deployments
3. Create custom dashboards in Grafana
4. Export the dashboards to code so that I can deploy them from scratch for new clusters

I didn't explore [AlertManager](https://prometheus.io/docs/alerting/alertmanager/) - but this would be essential for a complete monitoring solution. However, the building blocks are in place. I also found that "business telemetry" is difficult in Prometheus. By business telemetry I mean telemetry that has nothing to do with performance - things like how many products from category A were sold in a particular date range? AppInsights made "business telemetry" a breeze - the closest I could get in Prometheus was some proxy telemetry that gave me some idea of what was happening from a "business" perspective. Admittedly, Prometheus is a performance metric framework, so I wasn't too surprised.

### Security

There's a whole lot to security that I didn't fully explore - especially in-cluster Role Based Access Control (RBAC). What I did explore was how to secure your services using certificates - and how to do that declaratively and easily as you roll out a publicly accessible service. Again Traefik made this simple - I'll detail how I did it in my Traefik blogpost. As a corollary, I did also isolate "internal" from "external" services - the internal services are not accessible from outside the cluster at all - the simplest way to secure a service!

### Resiliency

I've already mentioned how using Deployments with rolling updates gives me zero-downtime deployments "for free". But what about throttling services that are being overwhelmed with connections? Or circuit-breakers? Again Traefik came to the rescue - and again, details are coming up in a post dedicated to how I configured Traefik.

### Infrastructure as Code

It (almost) goes without saying that all of these capabilities should be doable from scripts and templates - and that there shouldn't be any manual steps. I did that from the first - using [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest) scripts to spin up my [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/) (AKS) clusters, and configure Public IPs and Load Balancers. I used some bash scripts for doing kubectl commands and finally used [Helm](https://helm.sh/) for packaging my applications so that deployment is a breeze - including creating Ingresses, ServiceMonitors, Deployments, Secrets and all the pieces you need to run your services.

## Conclusion

I know I haven't showed very much - but I've gotten all of the pieces in place and will be blogging about how I did it - and sharing the code too! The point is that Container DevOps is more than just building images - there is far involved to do mature DevOps, and it's possible to achieve all of these mature practices using k8s. Traefik is definitely the star of the show! For now, I've hopefully prodded you into thinking about how to do some of these practices yourself.

Happy deploying!

