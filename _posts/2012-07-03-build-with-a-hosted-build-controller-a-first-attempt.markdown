---
layout: post
title: 'Build with a Hosted Build Controller: A First Attempt'
date: '2012-07-03 17:48:00'
tags:
- build
---

I am working on some code for the [TFS Tester Power Tool](http://visualstudiogallery.msdn.microsoft.com/72576517-821b-46c2-aa1a-fab940752292) with my colleague [Anna Russo](http://www.improvingsoftwarequality.com/) (who just got her first MVP award!) and we’re using TFS Preview for source control and work item tracking. From the start I wanted to get some unit tests and builds up and running. The challenge for the unit testing side was that the tool works against a Team Foundation Server, so testing required some sort of mocking or faking.

At around the same time I started looking at the unit tests, Microsoft released a beta of the [Fakes Framework](http://msdn.microsoft.com/en-us/library/tfs/hh549175(v=vs.110).aspx). I liked the look of the framework (especially since I had done some stuff with its predecessor, Moles), and decided to write the tests using it. However, I hit a snag with a [bug in the beta](http://colinsalmcorner.blogspot.com/2012/04/more-on-fakes-beta-has-issues.html). Fortunately, the [RC fixed the issue](http://colinsalmcorner.blogspot.com/2012/06/vs-2012-rc-fakes-bugs-fixed.html), and so I was able to get the tests running.

So once Anna had granted me EditBuildDefinition rights on the hosted Team Project, I was ready to spin my first hosted build.

Here are my experiences.

## Cloaked Build Drops folder
<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/--x0jdTD6PuQ/T_KxvqsJiHI/AAAAAAAAAZg/y-PmP57mtLk/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-zFldcgQM414/T_Kxsm-tjlI/AAAAAAAAAZY/hPVamI148dc/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

I never check-out my entire source control folder for a build (and neither should you!). When creating a build, make sure the workspace is the minimum codebase that you need. Since I didn’t have a solution open, the workspace defaulted to the root of the Team Project, and you can see that it adds a cloak for the source controlled drops folder. At first I was a little puzzled at this, and then I realised that if you do check out your entire repository for your build, you probably don’t need the previous build outputs, so this default makes sense. However, when I changed the root folder to a subfolder a bit deeper in, and didn’t delete the cloak of the drops folder – the build failed since the cloak didn’t have an active parent folder. So I just deleted the cloaked entry.

## Symbols
<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/--cSWlRr8E6U/T_Kxxs9vKhI/AAAAAAAAAZw/sr-SW1uEWhU/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-OV4n_yWmn38/T_KxwtUwHlI/AAAAAAAAAZo/DzC6GlYBG-s/s1600-h/image%25255B7%25255D.png)<!--kg-card-end: html-->

I like that you can copy the output of the build to a source controlled folder – this makes sense for TFS in the cloud. However, I tried to then add a source control folder as the destination for the symbols, and of course the build failed since the symbols target must be a valid UNC (I wonder if there’s a way to get the symbols into Source Control…).

## Hosted Build Queue Summary

I like the Queue Summary page that comes up when you queue a 2012 build – it tells you how long the average build time for this build definition is and where your build is in the queue.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-Su-A8GAGPx8/T_Kx08yoWYI/AAAAAAAAAaA/MYFd3aCaCqk/image_thumb%25255B10%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-qBj8hfLL79o/T_Kxz6B-XcI/AAAAAAAAAZ4/jwTDHxqC_sc/s1600-h/image%25255B22%25255D.png)<!--kg-card-end: html-->
## Enabling Code Coverage

I think that if you have unit tests, you may as well enable code coverage. Enabling code coverage in a 2012 build is a little different than it was in 2010 (where you did it via a runsettings file). In a 2012 build (this applies to on-premises as well as hosted), you need to edit the “Automated Tests” parameter of the build (press the ellipsis button in the right-hand column for this parameter). In the “General” tab, you’ll see a drop-down under “Options” that you can select to enable code coverage.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-nt9nCLmJcjQ/T_Kx2842D8I/AAAAAAAAAaQ/tputL10JR0Q/image_thumb%25255B6%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-vuIiMfzmijw/T_Kx18Vz5uI/AAAAAAAAAaI/AmJIOXVXnIY/s1600-h/image%25255B14%25255D.png)<!--kg-card-end: html-->
## Parameter name: Input (Value cannot be null)

So I ran the build again and I bumped into a strange error.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-_OkJVhkUuRw/T_Kx5Z7N1QI/AAAAAAAAAag/qTxGfxBhIY8/image_thumb%25255B8%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-hHJqOokv75c/T_Kx3gy8i-I/AAAAAAAAAaY/jo2VHctJT1o/s1600-h/image%25255B18%25255D.png)<!--kg-card-end: html-->

The error text wasn’t too helpful, so I queued the build again and changed the logging level to “Diagnostic”. That wasn’t too helpful either, except that I could see the error was happening in the catch of the Try-Catch around the testing activities.

After digging around in my ChampsList mail (one of the advantages of being an MVP) I came across Neno Loje’s mail about the same issue. Turns out there was a bug in the workflow which was fixed – make sure you use the DefaultTemplate11.1.xaml (as opposed to DefaultTemplate11.xaml).

## Code Coverage and Fakes – not friends

So now I noticed that the tests failed – even though I’ve run them on a local on-premises TeamBuild with no issues. I disabled Code Coverage, and the tests pass! So it seems at this point in time I can get the tests to pass, as long as I don’t enable Code Coverage. I’ve mailed the ChampsList, so hopefully I’ll get a resolution to this soon.

**Update:** see the follow up to this in [this post](http://colinsalmcorner.blogspot.com/2012/07/code-coverage-doesnt-work-with-fakes-on.html).

## Summary

All in all, it wasn’t too different getting a hosted build to work than getting an on-premises build to work. Once I get some resolution to the code coverage issue, I’ll get exactly the same functionality (I just have to figure out how to handle debug symbols elegantly…).

Happy building!

