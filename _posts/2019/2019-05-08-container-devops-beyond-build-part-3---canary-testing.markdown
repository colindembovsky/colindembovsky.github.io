---
layout: post
title: 'Container DevOps: Beyond Build (Part 3) - Canary Testing'
date: '2019-05-08 09:36:56'
tags:
- docker
---

1. TOC
{:toc}

Series:

- [Part 1: Intro](/container-devops-beyond-build-part-1)
- [Part 2: Traefik Basics](/container-devops-beyond-build-part-2---traefik)
- Part 3: Canary Testing (this post)
- [Part 4: Telemetry with Prometheus](/container-devops-beyond-build-part-4---telemetry-with-prometheus)
- [Part 5: Prometheus Operator](/container-devops-beyond-build-part-5---prometheus-operator)

In my [previous post](/container-devops-beyond-build-part-2---traefik) I compared [Istio](https://istio.io/), [Linkerd](https://linkerd.io/) and [Traefik](https://traefik.io/) and motivated why I preferred Traefik for Container DevOps. I showed how I was able to spin up Traefik controllers - one for internal cluster traffic routing, one for external cluster in-bound traffic routing. With that foundation in place, I can easily implement canary testing - both for external endpoints as well as internal services.

## Canary Testing

What is canary testing (sometimes referred to as A/B testing)? This is a technique of "testing in production" where you shift a small portion of traffic to a new version of a service to ensure it is stable, or that it is meeting some sort of business requirement, in the case of hypothesis-driven development. This is an important technique because no matter how good your test and staging environments are, there's no place like production. Sure, you can test performance in a test/stage environment, but you can only ever test user behavior in production! Being able to trickle a small amount of traffic to a new service limits exposure.

However, a lot of teams that do use canary testing tend to use it just for proving that a service is stable. I think that they're missing a trick - namely, telemetry and "proving hypotheses". Without good telemetry, you're never going to unlock the true potential of canary testing. Think of your canary as an experiment - and make sure you have a means to measure the success (or failure) of that experiment - otherwise you're just pointlessly mixing chemicals. I'll cover monitoring and telemetry in another post.

## Traffic Shifting Using Label Selectors

You can do canary testing "natively" in Kubernetes (k8s) by using good [label selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/). Imagine you have service Foo and it has label selectors "app=foo". Any pods that you deploy (typically via [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) or [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)) that have the label "app=foo" get traffic routed to them when the service endpoint is targeted. Imagine you had a Deployment that spins up two replicas of a pod with labels "app=foo,version=1.0". Hitting the Service endpoint will cause k8s to route traffic between the two pods. Now you have a new version of the container image and you create a Deployment that spins up one pod with labels "app=foo,version=1.1". Now because all three pods match the Service label selector "app=foo" traffic is distributed between all three pods - you've effectively routed 33% of traffic to the new pod.

So far so good. But here's where things get tricky: say you're monitoring the pods and decide that version 1.1 is good to go - how do you "promote" it to production fully? You could update the labels on the original pods and remove "app=foo" - they'll no longer match and so now all traffic is going to the third version 1.1 pod. But now you only have one pod, where originally you had two. So you'd have to also scale the Deployment of version 1.1 to ensure it gets as many replicas as the original service. And now you have a Deployment that's missing some labels - so you'd have to dig to find out what those pods are.

Alternatively, you could just add "version=1.1" to the Service label selectors. Again you'd have to scale the version 1.1 Deployment, but at least you don't get "dangling pods". But what about deploying version 1.2? Now you have to remove the "version=1.1" label from the Service since just adding "app=foo" won't be good enough to get traffic onto pods with labels "app=foo,version=1.2".

And how would you go about testing traffic shifting of just 2%? You'd need to deploy 49 replicas of version 1.1 and a single version 1.2 just to get that percentage.

What it boils down to is that using label selectors proves to be too much cognitive load since you spend too much time juggling labels, and the dial is "too course" - you can't easily test traffic percentages lower that say 20% very easily. In contrast, if you use Traefik to do the traffic shifting, you get the added bonus of circuit breakers, SSL and other features too.

## Traffic Shifting Using Traefik

Let's see how we'd do traffic shifting using Traefik. Let's suppose that I've already deployed a Traefik controller with "ingressClass=traefik.external". To route traffic between two identical services (where the only difference between the services is the image version) I can create this ingress:

    kind: Ingress
    metadata:
    annotations:
      kubernetes.io/ingress.class: traefik-external
      traefik.ingress.kubernetes.io/service-weights: |
        partsunlimited-website-blue: 5%
        partsunlimited-website-green: 95%
    labels:
      app: partsunlimited-website
      name: partsunlimited-website
    spec:
      rules:
      - host: cdk8spu-dev.westus.cloudapp.azure.com
        http:
          paths:
          - backend:
              serviceName: partsunlimited-website-blue
              servicePort: http
            path: /site
          - backend:
              serviceName: partsunlimited-website-green
              servicePort: http
            path: /site

Notes:

- Line 1: the kind of resource in "Ingress" - nothing special about this, it's a native k8s Ingress resource
- Line 4: this is where we specify which IngressController should do the heavy lifting for this particular Ingress
- Lines 5-7: simple, intuitive and declarative - we want 5% of traffic to be routed to the "blue" Service
- Line 13: when inbound traffic has host "cdk8spu-dev.westus.cloudapp.azure.com" (the DNS for the LoadBalancer), then we want the ingress to use the following rules to direct the traffic
- Lines 16-23: we specify the backend Services and Ports that the Ingress should route to and can even specify custom paths to map different backends to different URL paths

### The Services

This assumes that we have two services: partsunlimited-website-blue and partsunlimted-website-green. In my case these are exactly the same service - they will sometimes just have pods on different versions of the images I'm building. Let's look at the services:

    apiVersion: v1
    kind: Service
    metadata:
      name: partsunlimited-website-blue
      labels:
        app: partsunlimited
        canary: blue
    annotations:
      traefik.backend.circuitbreaker: "NetworkErrorRatio() &gt; 0.2"
    spec:
      ...
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: partsunlimited-website-green
      labels:
        app: partsunlimited
        canary: green
    spec:
      ...

Notes:

- Lines 5-7, 17-19: these are out-of-box label selectors for services. There's the common "app" label and then a label for each canary "slot" that I have
- Lines 8-9: since I am using Traefik, I can easily create a circuit-breaker using the annotation. In this case, we instruct the controller to cease to send traffic to the blue service if its network failure rate rises above 20%
- The other lines are exactly what you would use for defining any k8s service

## Helm

Now that I've shown you how to define the ingress and the services, I can discuss how I actually deployed my services. If you use "native" k8s yml manifests, it can become difficult to manage all your resources. Imagine you have several services, configmaps, secrets, ingresses, ingress controllers, persistence volumes - you'd need to manage each type of resource. [Helm](https://helm.sh/) simplifies that task by "bundling" the related resources. That way "helm upgrade" gives you a single command to install or upgrade all the resources - and similarly, "helm status" and "helm delete" let you inspect or destroy the app and all its resources quickly. So I built a helm package for my application that included the Traefik plumbing.

### Challenges with Helm

It's not all roses and unicorns though - helm has some disadvantages. Firstly, there's [Tiller](https://helm.sh/docs/glossary/#tiller) - the "server side" component of helm. To use helm, you need to install Tiller on your k8s cluster, and give it some pretty beefy permissions. [Helm 3](https://github.com/helm/community/blob/master/helm-v3/000-helm-v3.md) is abandoning Tiller, so this should improve in the near future.

The other (more pertinent) challenge is the way helm performs upgrades. Let's have a look at a snippet of the values file that I have for my service - this file is used to override (or supply) values to an actual deployment:

    canaries:
      - name: blue
        replicaCount: 1
        weight: 20
        tag: 1.0.0.0
        annotations:
          traefik.backend.circuitbreaker: "NetworkErrorRatio() &gt; 0.2"
      - name: green
        replicaCount: 2
        weight: 80
        tag: 1.0.0.0
        annotations: {}

Notes:

- Line 4,10 - I define the weights for each canary. Helm injects this value into the Ingress resource.
- Lines 5,6 - I define annotations to apply to the service - in this case the Traefik circuit-breaker, but I could add others too

Initially, I wanted to do a deployment with "version=1.0.0.0" for both canaries, and then just run "helm upgrade --set-values canaries[0].imageTag=1.0.0.1" to update the version of the blue canary. However, helm doesn't work this way and so I have to supply all the values for the chart, rather than just the ones I want to update. In a pipeline, the version to deploy to the blue canary is the latest build number - but I have to calculate the green canary version number or it will be overwritten with "1.0.0.0" every time. It's not a big deal since I can work it out, but it would be nice if helm had a way to only update a single value and leave all other current values "as-is".

In the end, the ease of managing the entire application (encompassing all the resources) using helm outweighed the minor paper-cuts. I still highly recommend helm to manage app deployment - even if they're simple apps!

## Conclusion

Traffic shifting using Traefik is pretty easy - it's also intuitive since it's based on annotations and is specified over "native" k8s resources instead of having to rely on custom constructs or sidecars or other rule-language formats. This makes it an ideal tool for performing canary testing in k8s deployments.

Happy canary testing!

