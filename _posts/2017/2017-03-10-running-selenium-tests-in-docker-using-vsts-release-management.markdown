---
layout: post
title: Running Selenium Tests in Docker using VSTS Release Management
date: '2017-03-10 00:43:25'
tags:
- docker
- releasemanagement
---

The other day I was doing a POC to run some Selenium tests in a Release. I came across some Selenium docker images that I thought would be perfect – you can spin up a Selenium grid (or hub) container and then join as many node containers as you want to (the node container is where the tests will actually run). The really cool thing about the node containers is that the container is configured with a browser (there are images for Chrome and Firefox) meaning you don’t have to install and configure a browser or manually run Selenium to join the grid. Just fire up a couple containers and you’re ready to test!

The source code for this post is on [Github](https://github.com/colindembovsky/vsts-selenium-docker-tests).

Here’s a diagram of the components:

<!--kg-card-begin: html-->[![image](/assets/images/files/3d91a7fa-2c3e-4cdb-aa9c-54d9b6491fbd.png "image")](/assets/images/files/25d0d00c-4ccc-4026-900b-f50755bb9b6c.png)<!--kg-card-end: html-->
## The Tests

To code the tests, I use Selenium WebDriver. When it comes to instantiating a driver instance, I use the RemoteWebDriver class and pass in the Selenium Grid hub URL as well as the capabilities that I need for the test (including which browser to use) – see line 3:

    private void Test(ICapabilities capabilities)
    {
        var driver = new RemoteWebDriver(new Uri(HubUrl), capabilities);
        driver.Navigate().GoToUrl(BaseUrl);
        // other test steps here
    }
    
    [TestMethod]
    public void HomePage()
    {
        Test(DesiredCapabilities.Chrome());
        Test(DesiredCapabilities.Firefox());
    }

Line 4 includes a setting that is specific to the test – in this case the first page to navigate to.

When running this test, we need to be able to pass the environment specific values for the HubUrl and BaseUrl into the invocation. That’s where we can use a runsettings file.

### Test RunSettings

The runsettings file for this example is simple – it’s just XML and we’re just using the TestRunParameters element to set the properties:

    &lt;?xml version="1.0" encoding="utf-8" ?&gt;
    &lt;RunSettings&gt;
        &lt;TestRunParameters&gt;
            &lt;Parameter name="BaseUrl" value="http://bing.com" /&gt;
            &lt;Parameter name="HubUrl" value="http://localhost:4444/wd/hub" /&gt;
        &lt;/TestRunParameters&gt;
    &lt;/RunSettings&gt;

You can of course add other settings to the runsettings file for the other environment specific values you need to run your tests. To test the setting in VS, make sure to go to Test-\>Test Settings-\>Select Test Settings File and browse to your runsettings file.

## The Build

The build is really simple – in my case I just build the test project. Of course in the real world you’ll be building your application as well as the test assemblies. The key here is to ensure that you upload the test assemblies as well as the runsettings file to the drop (more on what’s in the runsettings file later). The runsettings file can be uploaded using two methods: either copy it using a Copy Files task into the artifact staging directory – or you can mark the file’s properties in the solution to “Copy Always” to ensure it’s copied to the bin folder when you compile. I’ve selected the latter option.

Here’s what the properties for the file look like in VS:

<!--kg-card-begin: html-->[![image](/assets/images/files/21e1beba-4284-457a-8a7b-ab3e696965d9.png "image")](/assets/images/files/637a3469-5954-4028-a336-2af270dd3d08.png)<!--kg-card-end: html-->

Here’s the build definition:

<!--kg-card-begin: html-->[![image](/assets/images/files/1bc5d71f-aa64-44ea-baa2-8782da1090d2.png "image")](/assets/images/files/c75aac1c-c4d9-4e97-8843-bbba84245a84.png)<!--kg-card-end: html-->
## The Docker Host

If you don’t have a docker host, the fastest way to get one is to spin it up in Azure using the Azure CLI – especially since that will create the certificates to secure the docker connection for you! If you’ve got a docker host already, you can skip this section – but you will need to know where the certs are for your host for later steps.

Here are the steps you need to take to do that (I did this all in my Windows Bash terminal):

1. Install node and npm
2. Install the azure-cli using “npm install –g azure-cli”
3. Run “azure login” and log in to your Azure account
4. Don’t forget to set your subscription if you have more than one
5. Create an Azure Resource Group using “azure group create \<name\> \<location\>”
6. Run “azure vm image list –l westus –p Canonical” to get a list of the Ubuntu images. Select the Urn of the image you want to base the VM on and store it – it will be something like “Canonical:UbuntuServer:16.04-LTS:16.04.201702240”. I’ve saved the value into $urn for the next command.
7. Run the azure vm docker create command – something like this:
<!--kg-card-begin: html--><font face="Courier New">azure vm docker create --data-disk-size 22 --vm-size "Standard_d1_v2" --image-urn $urn --admin-username vsts --admin-password $password --nic-name "cd-dockerhost-nic" --vnet-address-prefix "10.2.0.0/24" --vnet-name "cd-dockerhost-vnet" --vnet-subnet-address-prefix "10.2.0.0/24" --vnet-subnet-name "default" --public-ip-domain-name "cd-dockerhost"  --public-ip-name "cd-dockerhost-pip" --public-ip-allocationmethod "dynamic" --name "cd-dockerhost" --resource-group "cd-docker" --storage-account-name "cddockerstore" --location "westus" --os-type "Linux" --docker-cert-cn "cd-dockerhost.westus.cloudapp.azure.com"</font><!--kg-card-end: html-->

Here’s the run from within my bash terminal:

<!--kg-card-begin: html-->[![image](/assets/images/files/e5770c4a-f7b1-4d40-aae0-b807fa449e23.png "image")](/assets/images/files/1307331b-df06-40fa-a28e-8861f282e19d.png)<!--kg-card-end: html-->

Here’s the result in the Portal:

<!--kg-card-begin: html-->[![image](/assets/images/files/4a1bd38f-c704-45ae-b152-a7ddae0e32c8.png "image")](/assets/images/files/0c178c4b-f5d5-449e-a6e1-1e5636f5529f.png)<!--kg-card-end: html-->

Once the docker host is created, you’ll be able to log in using the certs that were created. To test it, run the following command:

<!--kg-card-begin: html--><font face="Courier New">docker -H $dockerhost --tls info</font><!--kg-card-end: html--><!--kg-card-begin: html-->[![SNAGHTML2b3ba0](/assets/images/files/e426e1bb-4e15-4d3a-bfa9-3be04d47aa63.png "SNAGHTML2b3ba0")](/assets/images/files/179e94fd-6868-4e0d-911c-5b6c39fd9e21.png)<!--kg-card-end: html-->

I’ve included the commands in a [fish](https://fishshell.com/) script [here](https://github.com/colindembovsky/vsts-selenium-docker-tests/blob/master/scripts/createDockerHost.fish).

### The docker-compose.yml

The plan is to run multiple containers – one for the Selenium Grid hub and any number of containers for however many nodes we want to run tests in. We can call docker run for each container, or we can be smart and use docker-compose!

Here’s the docker-compose.yml file:

    hub:
      image: selenium/hub
      ports:
        - "4444:4444"
      
    chrome-node:
      image: selenium/node-chrome
      links:
        - hub
    
    ff-node:
      image: selenium/node-firefox
      links:
        - hub
    

Here we define three containers – named hub, chrome-node and ff-node. For each container we specify what image should be used (this is the image that is passed to a docker run command). For the hub, we map the container port 4444 to the host port 4444. This is the only port that needs to be accessible outside the docker host. The node containers don’t need to map ports since we’re never going to target them directly. To connect the nodes to the hub, we simple use the links keyword and specify the name(s) of the containers we want to link to – in this case, we’re linking both nodes to the hub container. Internally, the node containers will use this link to wire themselves up to the hub – we don’t need to do any of that plumbing ourselves - really elegant!

## The Release

The release requires us to run docker commands to start a Selenium hub and then as many nodes as we need. You can install [this extension](https://marketplace.visualstudio.com/items?itemName=ms-vscs-rm.docker) from the marketplace to get docker tasks that you can use in build/release. Once the docker tasks get the containers running, we can run our tests, passing in the hub URL so that the Selenium tests hit the hub container, which will distribute the tests to the nodes based on the desired capabilities. Once the tests complete, we can optionally stop the containers.

### Define the Docker Endpoint

In order to run commands against the docker host from within the release, we’ll need to configure a docker endpoint. Once you’ve installed the docker extension from the marketplace, navigate to your team project and click the gear icon and select Services. Then add a new Docker Host service, entering your certificates:

<!--kg-card-begin: html-->[![image](/assets/images/files/faad0481-94e6-4ee8-9703-2bed37159ae6.png "image")](/assets/images/files/7c1234cf-a831-4827-96fc-fda2f3fd8246.png)<!--kg-card-end: html-->
### Docker VSTS Agent

We’re almost ready to create the release – but you need an agent that has the docker client installed so that it can run docker commands! The easiest way to do this – is to run the vsts agent docker image on your docker host. Here’s the command:

    docker -H $dockerhost --tls run --env VSTS_ACCOUNT=$vstsAcc --env VSTS_TOKEN=$pat --env VSTS_POOL=docker -it microsoft/vsts-agent

I am connecting this agent to a queue called docker – so I had to create that queue in my VSTS project. I wanted a separate queue because I want to use the docker agent to run the docker commands and then use the hosted agent to run the tests – since the tests need to run on Windows. Of course I could have just created a Windows VM with the agent and the docker bits – that way I could run the release on the single agent.

### The Release Definition

Create a new Release Definition and start from the empty template. Set the build to the build that contains your tests so that the tests become an artifact for the release. Conceptually, we want to spin up the Selenium containers for the test, run the tests and then (optionally) stop the containers. You also want to deploy your app, typically before you run your tests – I’ll skip the deployment steps for this post. You can do all three of these phases on a single agent – as long as the agent has docker (and docker-compose) installed and VS 2017 to run tests. Alternatively, you can do what I’m doing and create three separate phases – the docker commands run against a docker-enabled agent (the VSTS docker image that I we just got running) while the tests run off a Windows agent. Here’s what that looks like in a release:

<!--kg-card-begin: html-->[![image](/assets/images/files/e32df3b8-00b6-4a57-898e-8afe7d417363.png "image")](/assets/images/files/43793dbc-bffe-48c5-ba75-c9ba2cb74725.png)<!--kg-card-end: html-->

Here are the steps to get the release configured:

1. Create a new Release Definition and rename the release by clicking the pencil icon next to the name
2. Rename “Environment 1” to “Test” or whatever you want to call the environment
3. Add a “Run on agent” phase (click the dropdown next to the “Add Tasks” button)
4. Set the queue for that phase to “docker” (or whatever queue you are using for your docker-enabled agents)
<!--kg-card-begin: html-->[![image](/assets/images/files/9be0bed0-d4aa-49b5-8fa2-f85d988fcc6c.png "image")](/assets/images/files/413ccf0a-5a6a-4477-a81d-d807c3b0fc4d.png)<!--kg-card-end: html-->
1. In this phase, add a “Docker-compose” task and configure it as follows:
<!--kg-card-begin: html-->[![image](/assets/images/files/2e0114be-d110-474b-b8d6-b3bb55e47db6.png "image")](/assets/images/files/ee82ca0b-2d9a-480a-baef-e54639c71a77.png)<!--kg-card-end: html-->
1. Change the action to “Run service images” (this ends up calling docker-compose up)
2. Uncheck Build Images and check Run in Background
3. Set the Docker Host Connection
4. In the next phase, add tasks to deploy your app (I’m skipping these tasks for this post)
5. Add a VSTest task and configure it as follows:
<!--kg-card-begin: html-->[![image](/assets/images/files/154ff891-cb9e-4dc6-8fc3-01d85fa3cd61.png "image")](/assets/images/files/b37910bc-6cf2-42be-b0f3-a8f708432712.png)<!--kg-card-end: html-->
1. I’m using V2 of the Test Agent task
2. I update the Test Assemblies filter to find any assembly with UITest in the name
3. I point the Settings File to the runsettings file
4. I override the values for the HubUrl and BaseUrl using environment variables
5. Click the ellipses button on the Test environment and configure the variables, using the name of your docker host for the HubUrl (note also how the port is the port from the docker-compose.yml file):
<!--kg-card-begin: html-->[![image](/assets/images/files/6f263221-b3b4-4233-b0a3-da3bab7f6bd5.png "image")](/assets/images/files/85392ae0-41e8-46eb-ac18-aae5075780d6.png)<!--kg-card-end: html-->
1. In the third (optional) phase, I use another Docker Compose task to run docker-compose down to shut down the containers
<!--kg-card-begin: html-->[![image](/assets/images/files/b81be1bd-09a0-42ed-84f8-a43b95d3f6d9.png "image")](/assets/images/files/bf70ea38-90fa-4904-91dd-bd81b9663ab7.png)<!--kg-card-end: html-->
1. This time set the Action to “Run a Docker Compose command” and enter “down” for the Command
2. Again use the docker host connection

We can now queue and run the release!

My release is successful and I can see the tests in the Tests tab (don’t forget to change the Outcome filter to Passed – the grid defaults this to Failed):

<!--kg-card-begin: html-->[![image](/assets/images/files/302c3c26-3251-4c25-8922-cb9866bc3866.png "image")](/assets/images/files/5ddee09a-106f-4bb6-8cf9-cef95d08a19f.png)<!--kg-card-end: html-->
## Some Challenges

### Docker-compose SSL failures

I could not get the docker-compose task to work using the VSTS agent docker image. I kept getting certificate errors like this:

<!--kg-card-begin: html--><font face="Courier New">SSL error: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:581)</font><!--kg-card-end: html-->

I did [log an issue](https://github.com/Microsoft/vsts-docker/issues/38) on the VSTS Docker Tasks repo, but I’m not sure if this is a bug in the extension or the VSTS docker agent. I was able to replicate this behavior locally by running docker-compose. What I found is that I can run docker-compose successfully if I explicitly pass in the ca.pem, cert.pem and key.pem files as command arguments – but if I specified them using environment variables, docker-compose failes with the SSL error. I was able to run docker commands successfully using the Docker tasks in the release – but that would mean running three commands (assuming I only want three containers) in the pre-test phase and another three in the post-test phase to stop each container. Here’s what that would look like:

<!--kg-card-begin: html-->[![image](/assets/images/files/c92d14c5-ff6c-46a3-af19-2d6ad0ade7e4.png "image")](/assets/images/files/b55b6e90-9922-4986-b8bb-bbab1ae00767.png)<!--kg-card-end: html-->

You can use the following commands to run the containers and link them (manually doing what the docker-compose.yml file does):

<!--kg-card-begin: html--><font face="Courier New">run -d -P --name selenium-hub selenium/hub</font><!--kg-card-end: html--><!--kg-card-begin: html--><font face="Courier New">run -d --link selenium-hub:hub selenium/node-chrome</font><!--kg-card-end: html--><!--kg-card-begin: html--><font face="Courier New">run -d --link selenium-hub:hub selenium/node-firefox</font><!--kg-card-end: html-->

To get the run for this post working, I just ran the docker-compose from my local machine (passing in the certs explicitly) and disabled the Docker Compose task in my release.

EDIT (3/9/2017): I figured out the issue I was having: when I created the docker host I wasn’t specifying a CN for the certificates. The default is \*, which was causing my SSL issues. When I configured the CN correctly using

--docker-cert-cn” "cd-dockerhost.westus.cloudapp.azure.com", everything worked nicely.

### Running Tests in the Hosted Agent

I also could not get the test task to run successfully using the hosted agent – but it did run successfully if I used a private windows agent. This is because at this time VS 2017 is not yet installed on the hosted agent. Running tests from the hosted agent will work just fine once VS 2017 is installed onto it.

## Pros and Cons

This technique is quite elegant – but there are pros and cons.

Pros:

- Get lots of Selenium nodes registered to a Selenium hub to enable lots of parallel testing (refer to my previous blog on how to [run tests in parallel in a grid](/parallel-testing-in-a-selenium-grid-with-vsts))
- No config required – you can run tests on the nodes as-is

Cons:

- Only Chrome and Firefox tests supported, since there are only docker images for these browsers. Technically you could join any node you want to to the hub container if you wanted other browsers, but at that point you may as well configure the hub outside docker anyway.

## Conclusion

I really like how easy it is to get a Selenium grid up and running using Docker. This should make testing fast – especially if you’re running tests in parallel. Once again VSTS makes advanced pipelines easy to tame!

Happy testing!

