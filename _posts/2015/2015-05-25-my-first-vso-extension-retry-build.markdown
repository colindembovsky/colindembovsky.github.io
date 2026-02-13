---
layout: post
title: 'My First VSO Extension: Retry Build'
date: '2015-05-25 17:58:53'
tags:
- tfsapi
---

Visual Studio Online (VSO) and TFS 2015 keep getting better and better. One of the coolest features to surface recently is the ability to add (supported) [extensions to VSO](https://www.visualstudio.com/en-us/integrate/extensions/overview). My good friend Tiago Pascoal managed to [hack VSO to add extensions](http://pascoal.net/task-board-enhancer/) a while ago, but it was achieved via browser extensions, not through a supported VSO extensibility framework. Now Tiago can add his extensions in an official manner!

TL;DR – if you just want the code for the extension, then just go to [this repo](https://github.com/colindembovsky/vso-colinsalmcorner-extensions).

<figure class="kg-card kg-image-card"><img src="https://github.com/colindembovsky/vso-colinsalmcorner-extensions/blob/master/src/retry-build/images/retry-build-screenshot.png?raw=true" class="kg-image" alt="retry-build-screenshot.png" loading="lazy"></figure>
## Retry Build

I was recently [playing with Build VNext](/why-you-should-switch-to-build-vnext) and got a little frustrated that there was no way to retry a build from the list of completed builds in Web Access. I had to click the build definition to queue it. I found this strange, since the build explorer in Visual Studio has an option to retry a build. I was half-way through writing a mail to the Visual Studio Product team suggesting that they add this option, when I had an epiphany: I can write that as an extension! So I did…

I started by browsing to the [Visual Studio Extensions sample repo](https://github.com/Microsoft/vso-extension-samples) on Github. I had to join the [Visual Studio Partner program](http://www.vsipprogram.com/join), which took a while since I signed up using my email address but adding my work Visual Studio account (instead of my personal account). Switching the account proved troublesome, but I was able to get it sorted with help from Will Smythe on the Product team. Make sure you’re the account owner and that you specify the correct VSO account when you sign up for the partner program!

Next I cloned the repo and took a look at the code – it looked fairly straightforward, especially since all I wanted to do with this extension was add a menu command – no new UI at all.

I followed the instructions for installing the “Contribution Point Guide” so that I could test that extensions worked on my account, as well as actually see the properties of the extension points. It’s a very useful extension to have when you’re writing extensions (does that sounds recursive?).

### TypeScript

I’m a huge TypeScript fan, so I wanted to write my extension in TypeScript. There is a sample in the samples repo that has TypeScript, so I got some hints from that. There is a “Delete branch” sample that adds a menu command (really the only thing I wanted to do), so I started from that sample and wrote my extension.

Immediately I was glad I had decided to use TypeScript – the d.ts (definition files) for the extension frameworks and services is very cool – getting IntelliSense and being able to type the objects that were passed around made discovery of the landscape a lot quicker than if I was just using plain JavaScript.

The code tuned out to be easy enough. However, when I ran the extension, I kept getting a

<!--kg-card-begin: html--><font face="Courier New">’define’ is not defined</font><!--kg-card-end: html-->

error. We’ll come back to that. Let’s first look at main.ts to see the extension:

    import {BuildHttpClient} from "TFS/Build/RestClient";
    import {getCollectionClient} from "VSS/Service";
    var retryBuildMenu = (function () {
        "use strict";
    
        return &lt;IContributedMenuSource&gt; {
            execute: (actionContext: any) =&gt; {
                var vsoContext = VSS.getWebContext();
                var buildClient = getCollectionClient(BuildHttpClient);
    
                VSS.ready(() =&gt; {
                    // get the build
                    buildClient.getBuild(actionContext.id, vsoContext.project.name).then(build =&gt; {
                        // and queue it again
                        buildClient.queueBuild(build, build.definition.project.id).then(newBuild =&gt; {
                            // and navigate to the build summary page
                            // e.g. https://myproject.visualstudio.com/DefaultCollection/someproject/_BuildvNext#_a=summary&amp;buildId=1347
                            var buildPageUrl = `${vsoContext.host.uri}/${vsoContext.project.name}/_BuildvNext#_a=summary&amp;buildId=${newBuild.id}`;
                            window.parent.location.href = buildPageUrl;
                        });
                    });
                });
            }
        };
    }());
    
    VSS.register("retryBuildMenu", retryBuildMenu);

Notes:

1. Lines 1/2: imports of framework objects – these 2 lines were causing an error for me initially
2. Line 3: the name of this function is only used on line 27 when we register it
3. Line 6: we’re returning an IContributedMenuSource struct
4. Line 7: the struct has an ‘execute’ method that is invoked when the user clicks on the menu item
5. Line 9: we get a reference to what is essentially the build service
6. Line 13: using the build it (a property I discovered on the actionContext object using the Contribution Point sample extension) we can get the completed build object
7. Line 15: I simple pop the build back onto the queue – all the other information is already in the build object (like branch, configuration and so on) from the previous queuing of the build
8. Line 18: I build an url that points to the summary page for the new build
9. Line 19: redirect the browser to the new build url
10. Line 13/15: note the use of the .then() syntax – these methods return promises (good asynch programming), so we use the .then() to execute once the async operation has completed
11. Line 27: registering the extension using the name (1st arg) which is the name we use in the extension.json file, and the function name we specified on line 3 (the 2nd arg)

It was in fact, simpler than I thought it would be. I was expecting to have to create a new build object from the old build object – turns out that wasn’t necessary at all. So I had my code and was ready to run – except that I ran into a snag. When I ran my code, I kept getting

<!--kg-card-begin: html--><font face="Courier New">’define’ is not defined</font><!--kg-card-end: html-->

. To understand why, we need to quickly understand how the extensions are organized.

## Anatomy of an Extension

A VSO extension consists of a couple of key files: the extension.json, the main.html and the main.ts or main.js file.

- extension.json – the manifest file for the extension – used to register the extension
- main.html – the main loading page for the extension – used to bootstrap the extension
- main.js (or main.ts) – the main script entry point for the extension – used to provide the starting point for any extension logic

The “Build Inspector” sample has a main.ts, but this file doesn’t really do much – it only redirects to the main page of the extension custom UI. So there are no imports or requires. So I was at a bit of a loss as to why I was getting what looked like a require error when my extension was loaded. Here’s the html for the main.html page of the sample “Delete Branch” extension:

    &lt;!DOCTYPE html&gt;
    &lt;html lang="en"&gt;
    &lt;head&gt;
        &lt;meta charset="UTF-8"&gt;
        &lt;title&gt;Delete Branch&lt;/title&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;script src="sdk/scripts/VSS.SDK.js"&gt;&lt;/script&gt;
        &lt;script src="scripts/main.js"&gt;&lt;/script&gt;
        &lt;script&gt;
            VSS.init({ setupModuleLoader: true });
        &lt;/script&gt;
    &lt;/body&gt;
    &lt;/html&gt;
    

You’ll see that the main.js file is imported in line 9, and then we’ve told VSO to use the module loader – necessary for any “require” work. So I was still baffled – here we’re telling the framework that we’re going to be using “require” and I’m getting a require error! (Remember, since the sample doesn’t use any requires in the main.js, it doesn’t error). My main.html page looked exactly the same – and then looked at the items.html page of the sample “Build Inspector” extension, and I got an idea – I need to require my main module, not just load it. Here’s what my main.html ended up looking like:

    &lt;!DOCTYPE html&gt;
    &lt;html lang="en"&gt;
    &lt;head&gt;
        &lt;meta charset="UTF-8"&gt;
        &lt;title&gt;Retry Build&lt;/title&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;script src="sdk/scripts/VSS.SDK.js"&gt;&lt;/script&gt;
        &lt;p&gt;User will never see this&lt;/p&gt;
        &lt;script type="text/javascript"&gt;
            // Initialize the VSS sdk
            VSS.init({
                setupModuleLoader: true,
            });
    
            // Wait for the SDK to be initialized, then require the script
            VSS.ready(function () {
                require(["scripts/main"], function (main) { });
            });
        &lt;/script&gt;
    &lt;/body&gt;
    &lt;/html&gt;
    

You can see how instead of just importing the main.js script (like the “Delete Branch” sample) I “require” the main script on line 18. Once I had that, no more errors and I was able to get the extension to work.

Once I had that worked out, I was able to quickly publish the extension to Azure, change the url in the extension.json file to point to my Azure site url, and I was done! The code is in [this repo](https://github.com/colindembovsky/vso-colinsalmcorner-extensions).

## Conclusion

Writing extensions for VSO is fun, and having a good sample library to start from is great. The “Contribution Points” sample is clever – letting you test the extension loading as well as giving very detailed information about the various hooks and properties available for extensions. Finally, the TypeScript definitions make navigating the APIs available a snap. While my first extension is rather basic, I am really pleased with the extensibility framework that the Product team have devised.

Happy customizing!

