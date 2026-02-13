---
layout: post
title: PaaS and Time Compression in the Cloud
date: '2015-06-22 16:31:08'
tags:
- cloud
---

Recently I got to write a couple of articles which were posted on the [Northwest Cadence Blog](http://blog.nwcadence.com/). I am not going to reproduce them here, so please read them on the NWCadence blog from the links below.

## PaaS

The first, [PaaS Architecture: Designing Apps for the Cloud](http://blog.nwcadence.com/paas-architecture-designing-for-the-cloud/) covers some of the considerations you’ll need to make if you’re porting your applications to the Cloud. Moving your website to a VM with IIS is technically “moving to the cloud”, but this is IaaS (infrastructure as a service) rather than PaaS (platform as a service). If you’re going to unlock true scale from the cloud, you’re going to have to move towards PaaS.

Unfortunately, you can’t simply push any site into PaaS, since you’ll need to consider where and how your data is stored, how your authentication is going to work, and most importantly, how your application will handle scale up – being spread over numerous servers simultaneously. The article deals with some these and other considerations you’ll need to make.

## Time Compression

The second article, [Compressing Time: A Competitive Advantage](http://blog.nwcadence.com/compressing-time-a-competitive-advantage/) is written off the back of Joe Weinman’s excellent white paper [Time for the Cloud](https://azureinfo.microsoft.com/rs/microsoftdemandcenter/images/EN-CNTNT-Whitepaper-Timeforthecloud.pdf?mkt_tok=3RkMMJWWfF9wsRokuKvKZKXonjHpfsX97%2BUsXa%2BwlMI%2F0ER3fOvrPUfGjI4CTcFlI%2BSLDwEYGJlv6SgFS7XCMadx37gOUxM%3D). Weinman asserts that “moving to the cloud” is not a guarantee of success in and of itself – companies must strategically utilize the advantages cloud computing offers. While there are many cloud computing advantages, Weinman focuses on what he calls _time compression_ – the cloud’s ability to speed _time to market_ as well as _time to scale_. Again, I consider some of the implications you’ll need to be aware of when you’re moving applications to the cloud.

Happy cloud computing!

