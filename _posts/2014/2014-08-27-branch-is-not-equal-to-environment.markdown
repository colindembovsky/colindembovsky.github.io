---
layout: post
title: 'Branch Is Not Equal to Environment: CODE-PROD Branching Strategy'
date: '2014-08-27 23:34:05'
tags:
- devops
- development
- releasemanagement
- build
---

Over the last couple of months I’ve done several implementations and upgrades of TFS 2013. Most organizations I work with are not developing boxed software – they’re developing websites or apps for business. The major difference is that boxed software often has more than one version of a product “in production” – some customers will be on version 1.0 while others will be on version 2.0 and so on. In this model, branches for each major version, with hot-fix branches where necessary – are a good way to keep these code bases separate while still being able to merge bug fixes across versions. However, I generally find that this is overkill for a “product” that only ever has one version in production at any one time – like internal applications or websites.

In this case, a well-established branching model is Dev-Main-Live.

## Dev-Main-Live
<!--kg-card-begin: html-->[![image](/assets/images/files/c1c98eb0-32c0-4f38-ba58-0c1f0c8aade8.png "image")](/assets/images/files/d69693b1-37e9-4174-8398-156cf40bd367.png)<!--kg-card-end: html-->

Dev-Main-Live (or sometimes Dev-Integration-Prod or other variants) is a fairly common branching model – new development is performed on the Dev branch (with multiple developers coding simultaneously). When changes are to be tested, they are merged to Main. There code is tested in a test or UAT environment, and when testing is complete the changes are merged to Live before being deployed to production. This means that if there are production issues (what? we have bugs?!?) those can be fixed on the Live branch – thus they can be tested and deployed independently from the Dev code which may not be production ready.

There are some issues with this approach:

1. You shouldn’t be taking so long to test that you need a separate Main branch. I only advise this for extensive test cycles – but you should be aiming to shorten your test cycles anyway. This makes the Main branch fairly obsolete – I’ve seen teams who always “merge through” Main to get changes from Dev to Live – so I’ve started advising getting rid of the Main branch altogether.
2. If you build code from Main, deploy it to Test and sign-off, you have to merge to Live before doing a build from the Live branch. This means that what you’re deploying isn’t what you tested (since you tested pre-merge). I’ve seen some teams deploy from the Main branch build, wait for several days, and then merge to the Live branch. Also a big no-no!
3. Usually bug fixes that are checked in on the Live branch don’t make it back to the Dev branch since you have to merge through Main – so the merge of new dev and bug fixes on the Live branch get done when Dev gets merged onto Live (through Main). This is too late in the cycle and can introduce merge bugs or rework.

This model _seems_ to work nicely since the branches “represent” the environments – what I have in Dev is in my dev environment, what’s on Main is in my Test environment and what’s in Live is in production, right? This “branch equals environment” mindset is actually hard to manage, so I’m starting to recommend a new approach.

## The Solution: Code-Prod with Builds
<!--kg-card-begin: html-->[![image](/assets/images/files/dadadd62-0806-4d08-ac1b-f7411074874c.png "image")](/assets/images/files/461eec0f-dc9e-4e5b-afc3-f5040958df71.png)<!--kg-card-end: html-->

So how should you manage code separation as well as know what code is in which environment at any time? The answer is to simplify the branching model and make use of builds.

In this scenario new development is done on the CODE branch (the name is to consciously separate the idea of the branch from the environment). When you’re ready to go to production, merge into PROD and do a build. The TFS build will (by default) label the code that is used to build the binaries. You’ll be able to tie the binary version to the build label if you use my [versioning script you can always match binaries to builds](/matching-binary-version-to-build-number-version-in-tfs-2013-builds). So you’ll be able to recreate a build, even if you lose the binaries somehow.

So now you have built “the bits” – notice how there is no mention of environment yet. You should be thinking of build and deploy as separate activities. Why? Because then you’ll be able to build a single package that can be deployed (and tested) in a number of environments. Of course you’re going to have to somehow manage configuration files for your different environments – for web projects you can refer to [my post](/webdeploy-and-release-management--the-proper-way) about how to parameterize the web.config so that you can deploy to any environment (the post is specific to Release Management, but the principles are the same for other deployment mechanisms and for any type of application that needs different configurations for different environments).

### Deployment – To Lab or To Release?

Let’s start off considering the “happy path” – you’ve done some coding in CODE, merged to PROD and produced a “production build”. It needs to be tested (of course you’ve already [unit tested](/why-you-absolutely-need-to-unit-test) as part of your build). Now you have two choices – Lab Management or Release Management. I like using a combination of Lab and Release, since each has a some good benefits. You can release to test using Lab Management (including automated deploy and test) so that your testers have an environment to test against – Lab Management allows rich data diagnostic collection during both automated and manual testing. You then use Release Management to get the bits into the release pipeline for deployment to UAT and Production environments, including automated deployment workflows and sign-offs. This way you only get builds into the release pipeline that have passed several quality gates (unit testing, automated UI testing and even manual testing) before getting into UAT. Irrespective of what approach you take, make sure you can take one build output and deploy it to multiple environments.

## But What About Bugs in Production?

If you get bugs in production **before** you do the merge, the solution is simple – fix the bug on the PROD branch, then build, test and release back to production. No messy untested dev CODE anywhere.

But what do you do if you have bugs after your merge, but before you’ve actually deployed to production? Hopefully you’re moving towards shorter release / test cycles, so this window should be short (and rare). But even if you do hit this scenario, there is a way to do the bug fix and keep untested code out. It’s a bit complicated (so you should be trying to avoid this scenario), but let me walk you through the scenario.

Let’s say we have a file in a web project called “Forecast.cs” that looks like this:

    public class Forecast
    {
        public int ID { get; set; }
    
        public DateTime Date { get; set; }
        public DayOfWeek Day 
        {
            get { return Date.DayOfWeek; } 
        }
    
        public int Min { get; set; }
    
        public int Max { get; set; }
    }

We’ve got a PROD build (1.0.0.4) and the label for 1.0.0.4 shows this file to be on version 51.

<!--kg-card-begin: html-->[![image](/assets/images/files/27816db5-0d10-4914-8a49-cdd85f993639.png "image")](/assets/images/files/0084ce46-7e76-48bf-9028-35997b93628b.png)<!--kg-card-end: html-->

We now make a change and add a property called “CODEProperty” (line 15) on the CODE branch:

    public class Forecast
    {
        public int ID { get; set; }
    
        public DateTime Date { get; set; }
        public DayOfWeek Day 
        {
            get { return Date.DayOfWeek; } 
        }
    
        public int Min { get; set; }
    
        public int Max { get; set; }
    
        public int CODEProperty { get; set; }
    }

We then check-in, merge to PROD and do another build (1.0.0.5). This version is then deployed out for testing in our UAT environment. Forecast.cs is now on version 53 in the 1.0.0.5 label, while all other files are on 51.

<!--kg-card-begin: html-->[![image](/assets/images/files/863f6109-a563-4bf5-9b16-b7dd5075e30e.png "image")](/assets/images/files/a10581e9-c553-4ce8-8e88-0fc7bc5a0a1b.png)<!--kg-card-end: html-->

Suddenly, the proverbial paw-paw hits the fan and there’s an urgent business-stopping bug in our currently deployed production version (1.0.0.4). So we go to source control, search for the 1.0.0.4 label in the PROD branch that the build created and select “Get This Version” to get the 1.0.0.4 version locally.

<!--kg-card-begin: html-->[![image](/assets/images/files/e7589b8f-bcb9-422c-9b9e-d730412407ae.png "image")](/assets/images/files/6242b8a1-2b86-4389-9d3f-b8c29459ba8a.png)<!--kg-card-end: html-->

We fix the bug (by adding a property called “HotfixProperty” – line 15 below). Note how there is no “CODEProperty” since this version of Forecast is before the CODEProperty checkin.

    public class Forecast
    {
        public int ID { get; set; }
    
        public DateTime Date { get; set; }
        public DayOfWeek Day 
        {
            get { return Date.DayOfWeek; } 
        }
    
        public int Min { get; set; }
    
        public int Max { get; set; }
    
        public int HotfixProperty { get; set; }
    }

Since we’re not on the latest version (we did a “Get-label”) we won’t be able to check in. So we shelve the change (calling the shelveset “1.0.0.4 Hotfix”). We then open the build template and edit the Get Version property and tell the build to get 1.0.0.4 too by specifying L followed by the label name – so the full “Get version” value is LPROD\_1.0.0.4:

<!--kg-card-begin: html-->[![image](/assets/images/files/ef8dd46d-ca85-4627-9f52-bf30daa455e4.png "image")](/assets/images/files/b247201f-a7d1-4e6b-8f99-3e3b312c86be.png)<!--kg-card-end: html-->

Next we queue the build, telling the build to apply the Shelveset too:

<!--kg-card-begin: html-->[![image](/assets/images/files/bee670af-da10-4178-8c78-19073e35ead5.png "image")](/assets/images/files/b91d162a-7920-4289-96f6-2d77f9f76596.png)<!--kg-card-end: html-->

We won’t be able to “Check in changes after successful build” since the build won’t be building with the Latest version. We’ll have to do that ourselves later. The build completes – we now have build 1.0.0.6 which can be deployed straight to production to “handle” the business-stopping bug.

Finally we do a Get Latest of the solution in PROD, unshelve the changeset to merge the Hotfix with the development code, clear the Get version property on the build and queue the next build that includes both the changes from CODE as well as the hotfix from PROD. This build is now 1.0.0.7. Meanwhile, testing is completed on 1.0.0.5, and so we can then fast-track the testing for 1.0.0.7 to release the new CODEProperty feature, including the hotfix from build 1.0.0.6.

Here’s a summary of what code is in what build:

- 1.0.0.4 – baseline PROD code
- 1.0.0.5 – CODEProperty change coming from a merge from CODE branch into PROD branch
- 1.0.0.6 – baseline PROD plus the hotfix shelveset (no CodeProperty at all) which includes the HotfixProperty
- 1.0.0.7 – CODEProperty merged with HotfixProperty

Here’s the 1.0.0.7 version of Forecast.cs (see lines 15 and 17):

    public class Forecast
    {
        public int ID { get; set; }
    
        public DateTime Date { get; set; }
        public DayOfWeek Day 
        {
            get { return Date.DayOfWeek; } 
        }
    
        public int Min { get; set; }
    
        public int Max { get; set; }
    
        public int CODEProperty { get; set; }
    
        public int HotfixProperty { get; set; }
    }

If we turn on Annotation, you’ll see that CODEProperty is changeset 52 (in green below), and HotfixProperty is changeset 54 (in red below):

<!--kg-card-begin: html-->[![image](/assets/images/files/5a65e063-be49-4938-81c8-5ec5b3f9b6df.png "image")](/assets/images/files/417517e7-1f8f-42fa-ba21-ae63060ec5e7.png)<!--kg-card-end: html-->

Yes, it’s a little convoluted, but it’ll work – the point is that this is possible without a 3rd branch in Source Control. Also, you should be aiming to shorten your test / release cycles so that this situation is very rare. If you hit this scenario often, you could introduce the 3rd branch (call it INTEGRATION or MAIN or something) that can be used to isolate bug-fixes in PROD from new development in CODE that isn’t ready to go out to production.

Here’s a summary of the steps if there is a bug in current production when you haven’t deployed the PROD code (after a merge from CODE) to production yet:

1. PROD code is built (1.0.0.4) and released to production.
2. CODE is merged to PROD and build 1.0.0.5 is created, but not deployed to production yet
3. Get by Label – the current PROD label (1.0.0.4)
4. Fix the bug and shelve your changes
5. Edit the build to change the Get version to the current PROD label (1.0.0.4)
6. Queue the build with your hotfix shelveset (this will be build 1.0.0.6)
7. Test and deploy the hotfix version (1.0.0.6) to production
8. Get Latest and unshelve to merge the CODE code and the hotfix
9. Clear the Get version field of the build and queue the new build (1.0.0.7)
10. Test and deploy to production

## Conclusion

The key to good separation of work streams is to not mistake the branch for the environment, nor confuse build with deploy. Using the CODE-PROD branching scenario, builds with versioning and labels, parameterized configs and Lab/Release management you can:

- Isolate development code from production code, so that you can do new features while still fixing bugs in production and not have untested development pollute the hotfixes
- Track which code is deployed where (using binary versions and labels)
- Recreate builds from labels
- Deploy a single build to multiple environments, so that what you test in UAT is what you deploy to production

Happy building and deploying!

