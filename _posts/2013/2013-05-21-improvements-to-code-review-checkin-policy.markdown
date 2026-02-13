---
layout: post
title: Improvements to Code Review Checkin Policy
date: '2013-05-21 00:33:00'
tags:
- alm
---

Late last year I uploaded my first VS Gallery contribution – [Colin’s ALM Corner Checkin Policies](http://visualstudiogallery.msdn.microsoft.com/c476b708-77a8-4065-b9d0-919ab688f078). One of the policies in this pack is a Code Review Checkin Policy. I blogged about it in [this post](http://www.colinsalmcorner.com/2012/12/custom-code-review-checkin-policy.html).

One of the pieces of feedback I got about this policy was that this is counter the “checkin early, checkin often” mantra of most development shops. Some suggestions were to only apply this policy to “junior” team members, allowing “senior” members to checkin without requiring a review. I decided however to approach this from a source control perspective as opposed to a group membership perspective.

The policy now allows you to configure which source control paths it must fire on – so if you checkin files in the “mapping”, a code review is required, otherwise no code review is necessary. This means you can make the DEV branch (or your equivalent) review-free, while enforcing reviews on MAIN or LIVE branches, according to your practices. This way you get the benefit of “checkin early, checkin often” without the hassle of reviewing every single change. When you merge a bunch of changes through to MAIN or LIVE, you can then enforce Code Review (you would perform your code review on the merge-set).

Here’s what the Policy Configuration settings now look like:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-r87J-QroDqY/UZpCWES-vmI/AAAAAAAAAus/E0SRc-OZ3_U/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-b8GGm8WncxY/UZpCUC-wWHI/AAAAAAAAAuk/rcUzkAhkcTM/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

You can see a list of “Paths to apply policy to”. Just add your source folders here and you get finer-grained control over when a Code Review is required.

Happy reviewing!

