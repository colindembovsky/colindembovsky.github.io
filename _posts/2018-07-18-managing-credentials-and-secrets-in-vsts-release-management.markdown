---
layout: post
title: Managing Credentials and Secrets in VSTS Release Management
date: '2018-07-18 02:36:55'
tags:
- tfsconfig
---

Releases almost always require some kind of credentials - from service credentials to database usernames and passwords. There are a number of ways to manage credentials in VSTS release management. In this post I'll look at a couple of common techniques. For brevity, I'm going to refer to _secrets_ as a proxy for secrets and credentials.

## Don't Store Secrets in Source Control

One bad practice you want to steer away from is storing secrets in source control. A lot of teams I work with have their build process create multiple environment-specific packages, using tools like [config transforms](https://msdn.microsoft.com/en-us/library/dd465318(v=vs.100).aspx). I like to get teams to think of build and release as two separate (but linked) processes:

**ProcessInputProcessOutput** BuildSource CodeCompile, unit test, packageTokenized build packagesReleaseBuild artifacts, config source codeInfrastructure deployment/config, approvals, integration/functional tests, app deploymentsDeployed application

The point is that the build should be _totally environment agnostic_. Good unit tests use mocking or fakes, so they shouldn't need environment-specific information. That means that they need to create packages that are, as I like to call them, swiss cheese - they need to have holes or tokens that can have environment-specific values injected at deployment time. You don't need tokens if your deployment process is capable of doing variable substitution - like the [IIS Deployment on Machine Group](https://docs.microsoft.com/en-us/vsts/pipelines/tasks/deploy/iis-web-app-deployment-on-machine-group?view=vsts) task or [Azure App Service Deployment](https://docs.microsoft.com/en-us/vsts/pipelines/tasks/deploy/azure-rm-web-app-deployment?view=vsts) task that can both do [inline variable replacement](https://docs.microsoft.com/en-us/vsts/pipelines/tasks/transforms-variable-substitution?view=vsts#xml-variable-substitution) (see my [earlier post](/easy-config-management-when-deploying-azure-web-apps-from-vsts) on how to do this - and this also now applies to the IIS Deployment on Machine Group task).

## Centralized vs Decentralized Secret Management

I see two broad categories of secret management: centralized and decentralized. Centralized secret management has the advantage of specifying/updating the secret once, even if it's used in many places - but has the disadvantage of being managed by a small subset of users (admins typically). This can also be an advantage, but can be a bottleneck. Decentralized secret management usually ends up in duplicated secrets (so updating a password leaves you hunting for every occurrence of that password) but removes the bottleneck of centralized management. Choosing a method will depend on your culture, auditing requirements and management overhead.

### Decentralized Secret Management

Decentralized secret management is the easiest to consider, and there's really only one way to do it: in your release definition, define your secrets as variables that are locked and you're done. If you need to use the same secret in multiple definitions, you just create the same variable. Of course if you change the value, you have to change it in each release that uses it. But you don't have to log a ticket or wait for anyone to change the value for you - if it changes, you update it in place for each release and you're done.

### Centralized Secret Management

There are three types of centralized secret management: [Azure KeyVault](https://azure.microsoft.com/en-us/services/key-vault/), [Variable Groups](https://docs.microsoft.com/en-us/vsts/pipelines/library/variable-groups?view=vsts) and Custom Key Vault. Let's consider each method.

The KeyVault and Variable Group methods both define a Variable Group - but if you use KeyVault, you manage the values in KeyVault rather than in the Variable Group itself. Otherwise they are exactly the same.

Go to the VSTS release hub and click on Library to see variable groups. Create a new Variable Group and give it a name. If this is a "plain" Variable Group, define all your secrets and their values - don't forget to padlock the values that you want to hide. If you're using KeyVault, first define a Service Endpoint in the Services hub for authenticating to the KeyVault. Then come back and link the Variable Group to the KeyVault and specify which Secrets are synchronized.

<!--kg-card-begin: html-->[![image](/assets/images/files/3e2a7993-9152-4c88-b60e-5447179db634.png "image")](/assets/images/files/e34bd373-bcc6-4049-a58c-8ec88c10a454.png)<!--kg-card-end: html-->

Now when you run define a release, you link the Variable Group (optionally scoping it) and voila - you have a centralized place to manage secrets, either directly in the Variable Group or via KeyVault.

<!--kg-card-begin: html-->[![image](/assets/images/files/81c1ba4e-e21a-438e-b4bb-150ce1d58cf2.png "image")](/assets/images/files/18eeb213-bcec-456c-b3d0-7e8de9957d68.png)<!--kg-card-end: html-->

The variable group can be linked to many releases, so you only ever have to manage the values in one place, irrespective of how many releases reference them. To use the values, just use $(SecretName) in your tasks.

The last method is Custom Key Vault. I worked with a customer a few months back that used some sort of third-party on-premises key vault. Fortunately this vault had a REST API and we were able to create a custom task that fetched secrets from this third-party key vault. If you do this, you need to remember to add in a custom task to get the values, but this was an elegant solution for my customer since they already had an internal key vault.

## Conclusion

There are a number of ways to manage secrets and credentials in VSTS/TFS. The most robust is to use Azure KeyVault, but if you don't have or don't want one you can use Variable Groups in-line. Whatever method you choose, just make sure you don't store any secrets in source control!

Happy releasing!

