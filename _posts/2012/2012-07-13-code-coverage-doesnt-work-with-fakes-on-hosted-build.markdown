---
layout: post
title: Code Coverage doesn’t work with Fakes on Hosted Build
date: '2012-07-13 19:01:00'
tags:
- build
---

In my [post about hosted build](http://colinsalmcorner.blogspot.com/2012/07/build-with-hosted-build-controller.html), I discovered that if you enable code coverage on unit tests that use the Fakes framework, the unit tests fail (even though the tests pass without code coverage turned on). The error is a “ShimNotSupportedException”.

I mailed the ChampsList, and it turns out that there is a problem with the hosted build servers for this scenario.In short, the problem has to do with the mix of RC and RTM on the build agent machines, which are running VS 2012 RC, and TfsPreview, that is running a later build (closer to RTM) of TFS.

When VS goes to RTM and the build agents are upgraded, this problem should go away. Until then, you’ll have to build the code on a local build server if you need code coverage and use the Fakes framework in your tests.

Happy building!

