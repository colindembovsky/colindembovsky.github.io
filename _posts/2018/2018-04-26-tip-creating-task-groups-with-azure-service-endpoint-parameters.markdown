---
layout: post
title: 'Tip: Creating Task Groups with Azure Service Endpoint Parameters'
date: '2018-04-26 02:03:43'
tags:
- releasemanagement
---

I've been working on some pretty complicated infrastructure deployment pipelines using my release management tool of choice (of course): VSTS Release Management. In this particular scenario, we're deploying a set of VMs to a region. We then want to deploy exactly the same setup but in a different region. Conceptually, this is like duplicating infrastructure between different datacenters.

Here's what the DEV environment in a release could look like:

<!--kg-card-begin: html-->[![image](/assets/images/files/7ed2a0f6-7ce1-484e-9330-be24aae46a5f.png "image")](/assets/images/files/ed25d298-ee8b-467e-a3e0-4dbddcce6284.png)<!--kg-card-end: html-->

If we're duplicating this to 5 regions, we'd need to clone the environment another 4 times. However, that would mean that any updates to any tasks would need to be duplicated over all 5 regions. It's easy to forget to update or to fat-finger a copy - isn't there a better way to maintain sets of tasks? I'm glad you asked…

### DRY - Don't Repeat Yourself

DRY (Don't Repeat Yourself) is a common coding practice - any time you find yourself copying code, you should extract it into a function so that you only have to maintain that logic in a single place. We can do the same thing in a release (or build) using Task Groups. Task Groups are like functions that you can call from releases (or builds) from many places - but maintain in a single place. Just like functions, they have parameters that you can set when you "call" them. Click the selector (checkmark icon to the right of each task) to select all the tasks you want to group, right-click and select "Create task group":

<!--kg-card-begin: html-->[![image](/assets/images/files/c08befdc-0ff9-48f2-ad4e-8cdb17451e2c.png "image")](/assets/images/files/fa1a36fe-da88-4587-92af-3ca1f3374b7e.png)<!--kg-card-end: html-->

A popup asks for the name of the Task Group and bubbles up all the parameters that are used in the tasks within the group. You can update the defaults and descriptions and click Create (helpful hint: make variables for all the values so that the variable becomes the default rather than a hard-coded value - this will make it easier to re-use the Task Group when you clone environments later):

<!--kg-card-begin: html-->[![image](/assets/images/files/d3ef4821-e41b-4942-805e-65c3bdad5ce0.png "image")](/assets/images/files/40bffa53-42ee-4477-b0e1-20c992792be8.png)<!--kg-card-end: html-->

So far, so good:

<!--kg-card-begin: html-->[![image](/assets/images/files/404885a7-146c-46b8-91da-cfb16b3a31cb.png "image")](/assets/images/files/746d677d-6ae8-4d5e-af7f-cb84fffd122e.png)<!--kg-card-end: html-->

However, there's a snag: looking at the parameters section, you'll notice that _we don't have any parameter for the Azure Service Endpoint_. Let's open the tasks and update the value in the dropdown to $(AzureSubscription):

<!--kg-card-begin: html-->[![image](/assets/images/files/afc66c70-15dd-4c4d-af1e-027d612c77ff.png "image")](/assets/images/files/074d6252-b72e-42a3-a73a-67c772f8b83e.png)<!--kg-card-end: html-->

Now you can see that the parameter is bubble up and surfaced as a parameter on the Task Group - it even has the dropdown with the Service Endpoints. Nice!

<!--kg-card-begin: html--> [![image](/assets/images/files/6c4508de-8141-4d2d-8323-d97a5aaccd08.png "image")](/assets/images/files/7cc4b761-099d-43b0-a3de-4c14fff9c8bf.png)<!--kg-card-end: html-->
### Consuming the Task Group

Open up the release again. You'll see that you now have a new parameter on the Task Group: the AzureSubscription. We'll select the DEV sub from the dropdown.

<!--kg-card-begin: html-->[![image](/assets/images/files/390bba92-c4bb-4645-b56c-a144a243d3cb.png "image")](/assets/images/files/ee0aee48-8a4c-4f0d-84e6-f038ad3a9659.png)<!--kg-card-end: html-->

Also note how the phase is now a single "task" (which is just a call to the Task Group). Under the hood, when the release is created, Release Management deletes the task group and replaces it with the tasks from the Task Group - so any values that are likely to change or be calculated on the fly should be variables.

Let's now clone the DEV environment to UAT-WESTUS and to UAT-EASTUS.

<!--kg-card-begin: html-->[![image](/assets/images/files/0862b9b8-ce28-4533-83d6-8f3257aa9674.png "image")](/assets/images/files/668ba059-3ca9-45d6-a3e4-a488c9d74234.png)<!--kg-card-end: html-->

If we edit the UAT-WESTUS, we can edit the service endpoint (and any other parameters) that we need to for this environment:

<!--kg-card-begin: html-->[![image](/assets/images/files/9836eed6-1101-4be4-a4b5-791a55eb78e5.png "image")](/assets/images/files/b0cf3e6c-30f7-4256-bfb6-4344e9e4b521.png)<!--kg-card-end: html-->

Excellent! Now we can update the Task Group in a single place even if we're using it in dozens of environments. Of course you'd need to update the other parameter values to have environment-specific values (Scopes) in the Variables section.

<!--kg-card-begin: html-->[![image](/assets/images/files/993e7370-8313-42f6-a3c2-87f0b355f9cb.png "image")](/assets/images/files/b0dce6d9-e3d0-45c9-abf5-7830a902e60d.png)<!--kg-card-end: html-->
## Conclusion

Task Groups are a great way to keep your releases (or builds) DRY - even allowing you to parameterize the Azure Service Endpoint so that you can duplicate infrastructure across different subscriptions or regions in Azure.

Happy deploying!

