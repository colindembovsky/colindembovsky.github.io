---
layout: post
title: 'New Task: Tag Build or Release'
date: '2017-05-02 12:49:12'
tags:
- development
- releasemanagement
---

I have a build/release task pack in the [marketplace](http://bit.ly/cacbuildtasks). I’ve just added a new task that allows you to add tags to builds or releases in the pipeline, inspired by my friend and fellow MVP Rene van Osnabrugge’s [excellent post](https://roadtoalm.com/2016/07/08/controlling-build-quality-using-build-tags-and-vsts-release-management/).

Here are a couple of use cases for this task:

1. You want to trigger releases, but only for builds on a particular branch with a particular tag. This trigger only works if the build is tagged _during the build_. So you could add a TagBuild task to your build that is only run [conditionally](https://www.visualstudio.com/en-us/docs/build/concepts/process/conditions) (for example for buildreason = Pull Request). Then if the condition is met, the tag is set on the build and the release will trigger in turn, but only for builds that have the tag set.
<!--kg-card-begin: html-->[![image](/assets/images/files/d9084b02-a33d-4834-96aa-9e66e4e77264.png "image")](/assets/images/files/b28ab04a-96bd-4a61-90ed-392f62ff5417.png)<!--kg-card-end: html-->
1. You want to tag a build from a release once a release gets to a certain environment. For example, you can add a TagBuild task and tag the primary build once all the integration tests have passed in the integration environment. That way you can see which builds have passed integration tests simply by querying the tags.
<!--kg-card-begin: html-->[![image](/assets/images/files/5a844571-d88c-407d-a4bd-da2f1a956e52.png "image")](/assets/images/files/1c80eeec-e5cd-46fa-b77a-c79adac0befa.png)<!--kg-card-end: html-->

Of course you can use variables for the tags – so you could tag the build with the release(s) that have made it to prod by specifying $(Release.ReleaseNumber) as the tag value.

There are of course a ton of other use cases!

### Tag Types

You can see the tag type matrix for the “tag type” (which can be set to Build or Release) in the [docs](https://github.com/colindembovsky/cols-agent-tasks/tree/master/Tasks/TagBuild).

### Conclusion

Let me know if you have issues or feedback. Otherwise, happy taggin’!

