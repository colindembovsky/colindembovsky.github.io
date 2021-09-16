---
layout: post
title: Gulp – Workaround for Handling VS Solution Configuration
date: '2015-01-16 19:30:54'
tags:
- development
---

We’ve got some [TypeScript](http://www.typescriptlang.org/) models for our web frontend. If you’re doing any enterprise JavaScript development, then TypeScript is a must. It’s much more maintainable and even gives you some compile-time checking.

Even though TypeScript is a subset of JavaScript, you still need to “compile” the TypeScript into regular JavaScript for your files to be used in a web application. Visual Studio 2013 does this out of the box (you may need to install TypeScript if you’ve never done so). However, what if you want to concat the compiled scripts to reduce requests from the site? Or minify them?

## WebEssentials

[WebEssentials](http://vswebessentials.com/) provides functionality like bundling and minification within Visual Studio. The MVC bundling features let you bundle your client side scripts “server side” – so the server takes a bundle file containing a list of files to bundle as well as some settings (like minification) and produces the script file “on-the-fly”. However, if you’re using some over framework, such as [Nancy](http://nancyfx.org/) – you’re out of luck. Well, not entirely.

## Gulp

What if you could have a pipeline (think: build engine) that could do some tasks such as concatenation and minification (and other tasks) on client-side scripts? Turns out there is a tool for that – it’s called [Gulp](http://gulpjs.com/) (there are several other tools in this space too). I’m not going to cover Gulp in much detail in this post – you can look to [Scott Hanselman’s excellent intro post](http://www.hanselman.com/blog/IntroducingGulpGruntBowerAndNpmSupportForVisualStudio.aspx). There are also some excellent tools for Visual Studio that support Gulp – notably [Task Runner Explorer Extension](https://visualstudiogallery.msdn.microsoft.com/8e1b4368-4afb-467a-bc13-9650572db708) (soon to be baked into VS 2015).

## Configurations for Gulp

After a bit of learning curve, we finally got our Gulp file into a good place – we were able to compile TypeScript to JavaScript, concat the files preserving ordering for dependencies, minify, include source maps and output to the correct directories. We even got this process kicked off as part of our TFS Team build for our web application.

However, I did run into a hitch – configurations. It’s easy enough to specify a configuration (or environment) for Gulp using the NODE\_ENV setting from the command line. Just set the value in the CLI you’re using (so “set NODE\_ENV Release” for DOS prompt, and “$env:NODE\_ENV = ‘Release’” for PowerShell) and invoke gulp. However, it seems that configurations are not yet supported within Visual Studio. I wanted to minify only for Release configurations – and I found there was no obvious way to do this.

I even managed to find a reply to a question on the Task Runner Explorer on VS Gallery where [Mads Krisensen](http://madskristensen.net/) states there is no configuration support for the extension yet – he says it’s coming though (see [here](https://visualstudiogallery.msdn.microsoft.com/8e1b4368-4afb-467a-bc13-9650572db708/view/Discussions/2) – look for the question titled “Passing build configuration into Gulp”).

The good news is I managed to find a passable workaround.

## The Workaround

In my gulp file I have the following lines:

    // set a variable telling us if we're building in release
    var isRelease = true;
    if (process.env.NODE_ENV &amp;&amp; process.env.NODE_ENV !== 'Release') {
        isRelease = false;
    }

This is supposed to grab the value of the NODE\_ENV from the environment for me. However, running within the Task Runner Explorer quickly showed me that it was not able to read this value from anywhere.

At the top of the Gulp file, there is a /// comment that allows you to bind VS events to the Gulp file – so if you want a task executed before a build, you can set BeforeBuild=’default’ inside the ///. At first I tried to set the environment using “set ENV\_NODE $(Configuration)” in the pre-build event for the project, but no dice.

Here’s the workaround:

- Remove the BeforeBuild binding from the Gulp file (i.e. when you build your solution, the Gulp is not triggered). You can see your bindings in the Task Runner Explorer – you want to make sure “Before Build” and “After Build” are both showing 0:
<!--kg-card-begin: html-->[![image](/assets/images/files/ffdc4f2a-2252-4913-a7d0-ad319441e07d.png "image")](/assets/images/files/52d9d0d7-57d9-4c1e-bd63-87e51fe08c0b.png)<!--kg-card-end: html-->
- Add the following into the Pre-Build event on your Web project Build Events tab:
- set NODE\_ENV=$(ConfigurationName)
- gulp
<figure class="kg-card kg-image-card"><img src="/assets/images/files/7708bf5c-ba1a-48b0-add0-2442734925a8.png" class="kg-image" alt="image" loading="lazy" title="image"></figure>

That’s it! Now you can just change your configuration from “Debug” to “Release” in the configuration dropdown in VS and when you build, Gulp will find the correct environment setting. Here you can see I set the config to Debug, the build executes in Debug and Gulp is correctly reading the configuration setting:

<!--kg-card-begin: html-->[![image](/assets/images/files/4ed9d16f-af16-4f65-9e4c-1c884c49472b.png "image")](/assets/images/files/b6e9a620-1c68-4d7b-a65a-705f3ec21f79.png)<!--kg-card-end: html-->
### Caveats

There are always some – in this case only two that I can think of:

1. You won’t see the Gulp output in the Task Runner Explorer Window on builds – however, if you use the Runner to invoke tasks yourself, you’ll still see the output. The Gulp output will now appear in the Build output console when you build.
2. If you’ve set a Watch task (to trigger Gulp when you change a TypeScript file, for example) it won’t read the environment setting. For me it’s not a big deal since build is invoked prior to debugging from VS anyway. Also, for our build process I default the value to “release” just in case.

Happy gulping!

