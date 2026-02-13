---
layout: post
title: Using Linked ARM Templates with VSTS Release Management
date: '2018-01-23 04:29:33'
tags:
- cloud
- releasemanagement
---

If you've ever had to create a complex ARM template, you'll know it can be a royal pain. You've probably been tempted to split out your giant template into smaller templates that you can link to, only to discover that you can only link to a sub-template if the sub-template is accessible via some public URI. Almost all of the examples in the [Template Quickstart repo](https://github.com/Azure/azure-quickstart-templates) that have links simply refer to the public Github URI of the linked template. But what if you want to refer to a private repo of templates?

## Using Blob Containers

The solution is to use blob containers. You upload the templates to a private container in an Azure Storage Account and then create a SAS token for the container. Then you create the full file URI using the container URI and the SAS token. Sounds simple, right? Fortunately with VSTS Release Management, it actually is easy.

As an example, let's look at this template that is used to create a VNet and some subnets. First we'll look at the VNet template (the linked template) and then how to refer to it from a parent template.

### The Child Template

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "vnetName": {
          "type": "string"
        },
        "vnetPrefix": {
          "type": "string"
        },
        "subnets": {
          "type": "object"
        }
      },
      "variables": {
      },
      "resources": [
        {
          "name": "[parameters('vnetName')]",
          "type": "Microsoft.Network/virtualNetworks",
          "location": "[resourceGroup().location]",
          "apiVersion": "2016-03-30",
          "dependsOn": [],
          "tags": {
            "displayName": "vnet"
          },
          "properties": {
            "addressSpace": {
              "addressPrefixes": [
                "[parameters('vnetPrefix')]"
              ]
            }
          }
        },
        {
          "apiVersion": "2015-06-15",
          "type": "Microsoft.Network/virtualNetworks/subnets",
          "tags": {
            "displayName": "Subnets"
          },
          "copy": {
            "name": "iterator",
            "count": "[length(parameters('subnets').settings)]"
          },
          "name": "[concat(parameters('vnetName'), '/', parameters('subnets').settings[copyIndex()].name)]",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "[parameters('vnetName')]"
          ],
          "properties": {
            "addressPrefix": "[parameters('subnets').settings[copyIndex()].prefix]"
          }
        }
      ],
      "outputs": {
      }
    }

Notes:

- There are 3 parameters: the VNet name and prefix (strings) and then an object that contains the subnet settings
- The first resource is the VNet itself - nothing complicated there
- The second resource uses copy to create 0 or more instances. In this case, we're looping over the subnets.settings array and creating a subnet for each element in that array, using copyIndex() as the index as we loop

There's really nothing special here - using a copy is slightly more advanced, and the subnets parameter is a complex object. Otherwise, this is plain ol' ARM json.

### The Parent Template

The parent template has two things that are different from "normal" templates: it needs two parameters (containerUri and containerSasToken) that let it refer to the linked (child) template and it invokes the template by specifying a "Microsoft.Resources/deployments" resource type. Let's look at an example:

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "containerUri": {
          "type": "string"
        },
        "containerSasToken": {
          "type": "string"
        }
      },
      "variables": {},
      "resources": [
        {
          "apiVersion": "2017-05-10",
          "name": "linkedTemplate",
          "type": "Microsoft.Resources/deployments",
          "properties": {
            "mode": "incremental",
            "templateLink": {
              "uri": "[concat(parameters('containerUri'), '/Resources/vNet.json', parameters('containerSasToken'))]",
              "contentVersion": "1.0.0.0"
            },
            "parameters": {
              "vnetName": { "value": "testVNet" },
              "vnetPrefix": { "value": "10.0.0.0/16" },
              "subnets": {
                "value": {
                  "settings": [
                    {
                      "name": "subnet1",
                      "prefix": "10.0.0.0/24"
                    },
                    {
                      "name": "subnet2",
                      "prefix": "10.0.1.0/24"
                    }
                  ]
                }
              }
            }
          }
        }
      ],
      "outputs": {}
    }

Notes:

- There are two parameters that pertain to the linked template: the containerUri and the SAS token
- In the resources, there is a "Microsoft.Resources/deployment" resource - this is how we invoke the child template
- In the templateLink, the URI is constructed by concatenating the containerUri, the path to the child template within the container, and the SAS token
- Parameters are passed inline - note that even simple parameters look like JSON objects (see vNetName and vnetPrefix)

Initially I tried to make the subnets object an array: but this blew up on the serialization. So I made an object called "settings" that is an array. So the subnets value property is an object called "settings" that is an array. You can look back at the child template to see how I dereference the object to get the values: to get the name of a subnet, I use "parameters('subnet').settings[index].name" (where index is 0 or 1 or whatever). The copy uses the length() method to get the number of elements in the array and then I can use copyIndex() to get the current index within the copy.

Of course the parent template can contain other resources - I just kept this example really simple to allow us to zoom in on the linking bits.

### Source Structure

Here's a look at how I laid out the files in the Azure Resource Group project:

<!--kg-card-begin: html-->[![image](/assets/images/files/70c8a31a-fe17-41ed-8319-a1db70eff95a.png "image")](/assets/images/files/d859a2fc-824c-4900-8029-6e6460e7874d.png)<!--kg-card-end: html-->

You can see how the vNet.json (the child template) is inside a folder called "Resources". I use that as the relative path when constructing the URI to the child template.

## The Release Definition

Now all the hard work is done! To get this into a release, we just create a storage account in Azure (that we can copy the templates to) and we're good to go.

Now create a new release definition. Add the repo containing the templates as a release artifact. Then in your environment, drop two tasks: Azure File Copy and Azure Resource Group Deployment. We configure the Azure File Copy task to copy all our files to the storage account into a container called templates. We also need to give the task two variable names: one for the containerUri and one for the SAS token:

<!--kg-card-begin: html-->[![image](/assets/images/files/8cab0747-bb3c-4265-a0ea-58193364e490.png "image")](/assets/images/files/ef6934b9-1161-4256-9e49-e7ecbb5f86b0.png)<!--kg-card-end: html-->

Once this task has executed, the templates will be available in the (private) container with the same folder structure as we have in Visual Studio.

On the next task, we can select the parent template as the template to invoke. We can pass in any parameters that are needed - at the very least, we need the containerUri and SAS token, so we pass in the variables from the previous task using $() notation:

<!--kg-card-begin: html-->[![image](/assets/images/files/136c37f0-a276-4851-8ad5-bfad48ed208c.png "image")](/assets/images/files/061159e7-1035-4924-b653-82cd7d7280cf.png)<!--kg-card-end: html-->

Now we can run the release and voila - we'll have a vNet with two subnets.

## Conclusion

Refactoring templates into linked templates is good practice - it's DRY (don't repeat yourself) and can make maintenance of complicated templates a lot easier. Using VSTS Release Management and a storage container, we can quickly, easily and securely make linked templates available and it all just works ™.

Happy deploying!

