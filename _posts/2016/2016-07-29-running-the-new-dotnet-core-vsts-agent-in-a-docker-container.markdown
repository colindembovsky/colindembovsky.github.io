---
layout: post
title: Running the New DotNet Core VSTS Agent in a Docker Container
date: '2016-07-29 01:12:06'
tags:
- docker
- build
---

This week I finally got around to updating my [VSTS extension](https://marketplace.visualstudio.com/items?itemName=colinsalmcorner.colinsalmcorner-buildtasks) (which bundle x-plat [VersionAssembly](https://github.com/colindembovsky/cols-agent-tasks/tree/master/Tasks/VersionAssemblies) and [ReplaceTokens](https://github.com/colindembovsky/cols-agent-tasks/tree/master/Tasks/ReplaceTokens) tasks) to use the new [vsts-task-lib](https://github.com/Microsoft/vsts-task-lib), which is used by the new [DotNet Core vsts-agent](https://github.com/Microsoft/vsts-agent/blob/master/README.md). One of the bonuses of the new agent is that it can run in a DotNet Core Docker container! Since I am running [Docker for Windows](https://docs.docker.com/docker-for-windows/), I can now (relatively) easily spin up a test agent in a container to run test – a precursor to running the agent in a container as the _de-facto_ method of running agents!

All you need to do this is a Dockerfile with a couple of commands that do the following:

1. Install Git
2. Create a non-root user and switch to it (since the agent won’t run as root)
3. Copy the agent tar.gz file and extract it
4. Configure the agent to connect it to VSTS

Pretty simple.

## The Dockerfile

Let’s take a look at the Dockerfile (which you can find [here in Github](https://github.com/colindembovsky/cols-agent-tasks/tree/master/docker)) for an agent container:

    FROM microsoft/dotnet:1.0.0-core
    
    # defaults - override them using --build-arg
    ARG AGENT_URL=://github.com/Microsoft/vsts-agent/releases/download/v2.104.0/vsts-agent-ubuntu.14.04-x64-2.104.0.tar.gz
    ARG AGENT_NAME=docker
    ARG AGENT_POOL=default
    
    # you must supply these to the build command using --build-arg
    ARG VSTS_ACC
    ARG PAT
    
    # install git
    #RUN apt-get update &amp;&amp; apt-get -y install software-properties-common &amp;&amp; apt-add-repository ppa:git-core/ppa
    RUN apt-get update &amp;&amp; apt-get -y install git
    
    # create a user
    RUN useradd -ms /bin/bash agent
    USER agent
    WORKDIR /home/agent
    
    # download the agent tarball
    #RUN curl -Lo agent.tar.gz $AGENT_URL &amp;&amp; tar xvf agent.tar.gz &amp;&amp; rm agent.tar.gz
    COPY *.tar.gz .
    RUN tar xzf *.tar.gz &amp;&amp; rm -f *.tar.gz
    RUN bin/Agent.Listener configure --url https://$VSTS_ACC.visualstudio.com --agent $AGENT_NAME --pool $AGENT_POOL --acceptteeeula --auth PAT --token $PAT --unattended
    
    ENTRYPOINT ./run.sh

Notes:

- Line 1: We start with the DotNet Core 1.0.0 image
- Lines 4-6: We create some arguments and set defaults
- Lines 9-10: We create some args that don’t have defaults
- Line 14: Install Git
- This installs Git 2.1.4 from the official Jesse packages. We should be installing Git 2.9, but the only way to install it from a package source is to add a package source (line 13, which I commented out). Unfortunately apt-add-repository is inside the package software-properties-common, which introduces a lot of bloat to the container which I decided against. The VSTS agent will work with Git 2.1.4 (at least at present) so I was happy to leave it at that.
- Line 17: create a user called agent
- Line 18: switch to the agent user
- Line 19: switch to the agent home directory
- Line 23: Use this to download the tarball as part of building the container. Do it if you have enough bandwidth. I ended up downloading the tarball and putting it in the same directory as the Dockerfile and using Line 24 to copy it to the container
- Line 24: Extract the tarball and then delete it
- Line 25: Run the command to configure the agent in an unattended mode. This uses the args supplied through the file or from the docker build command to correctly configure the agent.
- Line 27: Set an entrypoint – this is the command that will be executed when you run the container.

Pretty straightforward. To build the image, just cd to the Dockerfile folder and download the agent tarball (from [here](https://github.com/Microsoft/vsts-agent/releases)) if you’re going to use Line 23 (otherwise if you use Line 22, just make sure Line 4 has the latest release URL for Ubuntu 14.04 or use the AGENT\_URL arg to supply it when building the image). Then run the following command:

    docker build . --build-arg VSTS_ACC=myVSTSAcc --build-arg PAT=abd64… --build-arg AGENT_POOL=docker –t colin/agent

- Mandatory: VSTS\_ACC (which is the 1st part of your VSTS account URL – so for [https://myVSTSAcc.visualstudio.com](https://myVSTSAcc.visualstudio.com) the VSTS\_ACC is myVSTSAcc.
- Mandatory: PAT – your Personal Auth Token
- Optional: AGENT\_POOL – the name of the agent pool you want the agent to register with
- Optional: AGENT\_NAME – the name of the agent
- Optional: AGENT\_URL – the URL to the Ubuntu 14.04 agent (if using Line 22)
- The –t is the tag argument. I use colin/agent.

This creates a new image that is registered with your VSTS account!

Now that you have an image, you can simply run it whenever you need your agent:

    &gt; docker run -it colin/agent:latest
    
    Scanning for tool capabilities.
    Connecting to the server.
    2016-07-28 17:56:57Z: Listening for Jobs
    

After the docker run command, you should see the agent listening for jobs.

## Gotcha – Self-Updating Agent

One issue I did run into is that I had downloaded agent 2.104.0. When the first build runs, the agent checks to see if there’s a new version available. In my case, 2.104.1 was available, so the agent updated itself. It also restarts – however, if it’s running in a container, when the agent stops, the container stops. The build fails with this error message:

<!--kg-card-begin: html--><font face="Courier New">The job has been abandoned because agent docker did not renew the lock. Ensure agent is running, not sleeping, and has not lost communication with the service.</font><!--kg-card-end: html-->

Running the container again starts it with the older agent again, so you get into a loop. Here’s how to break the loop:

1. Run docker run -it --entrypoint=/bin/bash colin/agent:latest
2. This starts the container but just creates a prompt instead of starting the agent
3. In the container, run “./run.sh”. This will start the agent.
4. Start a build and wait for the agent to update. Check the version in the capabilities pane in the Agent Queue page in VSTS. The first build will fail with the above “renew lock” error.
5. Run a second build to make sure the agent is working correctly.
6. Now exit the container (by pressing Cntr-C and then typing exit).
7. Commit the container to a new image by running docker commit --change='ENTRYPOINT ./run.sh' \<containerId\> (you can get the containerId by running docker ps)
8. Now when you run the container using docker run –it colin/agent:latest your agent will start and will be the latest version. From there on, you’re golden!

## Conclusion

Overall, I was happy with how (relatively) easy it was to get an agent running in a container. I haven’t yet tested actually compiling a DotNet Core app – that’s my next exercise.

Happy Dockering!

