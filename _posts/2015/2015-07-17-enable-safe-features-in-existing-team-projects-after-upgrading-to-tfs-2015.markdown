---
layout: post
title: Enable SAFe Features in Existing Team Projects After Upgrading to TFS 2015
date: '2015-07-17 15:50:11'
tags:
- tfsconfig
- alm
---

TFS 2015 has almost reached RTM! If you upgrade to CTP2, you’ll see a ton of new features, not least of which are significant backlog and board improvements, the identity control, Team Project rename, expanded features for Basic users, the new Build Engine, PRs and policies for Git repos and more. Because of the schema changes required for Team Project rename, this update can take a while. If you have large databases, you may want to run the “pre-upgrade” utility that will allow you to prep your server while it’s still online and decrease the time required to do the upgrade (which will need to be done offline).

## SAFe Support

The three out of the box templates have been renamed to simply Scrum, Agile and CMMI. Along with the name change, there is now “built in” support for SAFe. This means if you create a new TFS 2015 team project, you’ll have 3 backlogs – Epic, Feature and “Requirement” (where Requirement will be Requirement, User Story or PBI for CMMI, Agile and Scrum respectively). In Team Settings, team can opt into any of the 3 backlogs. Also, Epics, Features and “Requirements” now have an additional “Value Area” field which can be Business or Architectural, allowing you to track Business vs Architectural work.

## Where are my Epics?

After upgrading my TFS to 2015, I noticed that I didn’t have Epics. I remember when upgrading from 2012 to 2013, when you browsed to the Backlog a message popped up saying, “Some features are not available” and a wizard walked you through enabling the “backlog feature”, adding in missing work items and configuring the process template settings. I was expecting the same behavior when upgrading to TFS 2015 – but that didn’t happen. I pinged the TFS product team and they told me that, “Epics are not really a new ‘feature’ per se – just a new backlog level, so the ‘upgrade’ ability was not built in.” If you’re on VSO, your template did get upgraded, so you won’t have a problem – however, for on-premises Team Projects you have to apply the changes manually.

### Doing it Manually

Here are the steps for enabling SAFe to your existing TFS 2013 Agile, Scrum or CMMI templates:

1. Add the Epic work item type
2. Add the “Value Area” field to Features and “Requirements”
3. Add the “Value Area” field to the Feature and “Requirement” form
4. Add the Epic category
5. Add the Epic Product Backlog
6. Set the Feature Product Backlog parent to Epic Backlog
7. Set the work item color for Epics

It’s a whole lot of “witadmin” and XML editing – never fun. Fortunately for you, I’ve created a script that will do it for you.

### Isn’t there a script for that?

Here’s the script – but you can download it from [here](http://1drv.ms/1T9Ph0q).

    &lt;#
    .SYNOPSIS
    
    Author: Colin Dembovsky (http://colinsalmcorner.com)
    Updates 2013 Templates to 2015 base templates, including addition of Epic Backlog and Area Value Field.
    
    
    .DESCRIPTION
    
    Adds SAFe support to the base templates. This involves adding the Epic work item (along with its backlog and color settings) as well as adding 'Value Area' field to Features and Requirements (or PBIs or User Stories).
    
    This isn't fully tested, so there may be issues depending on what customizations of the base templates you have already made. The script attempts to add in values, so it should work with your existing customizations.
    
    To execute this script, first download the Agile, Scrum or CMMI template from the Process Template Manager in Team Explorer. You need the Epic.xml file for this script.
    
    .PARAMETER tpcUri
    
    The URI to the Team Project Collection - defaults to 'http://localhost:8080/tfs/defaultcollection'
    
    .PARAMETER project
    
    The name of the Team Project to ugprade
    
    .PARAMETER baseTemplate
    
    The name of the base template. Must be Agile, Scrum or CMMI
    
    .PARAMETER pathToEpic
    
    The path to the WITD xml file for the Epic work item
    
    .PARAMETER layoutGroupToAddValueAreaControlTo
    
    The name of the control group to add the Value Area field to in the FORM - defaults to 'Classification' (Agile), 'Details' (SCRUM) and '' (CMMI). Leave this as $null unless you've customized your form layout.
    
    .PARAMETER pathToWitAdmin
    
    The path to witadmin.exe. Defaults to 'C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\witadmin.exe'
    
    .EXAMPLE
    
    Upgrade-Template -project FabrikamFiber -baseTemplate Agile -pathToEpic '.\Agile\WorkItem Tracking\TypeDefinitions\Epic.xml'
    
    #&gt;
    
    param(
        [string]$tpcUri = "http://localhost:8080/tfs/defaultcollection",
    
        [Parameter(Mandatory=$true)]
        [string]$project,
    
        [Parameter(Mandatory=$true)]
        [ValidateSet("Agile", "Scrum", "CMMI")]
        [string]$baseTemplate,
    
        [Parameter(Mandatory=$true)]
        [string]$pathToEpic,
    
        [string]$layoutGroupToAddValueAreaControlTo = $null,
    
        [string]$pathToWitAdmin = 'C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\witadmin.exe'
    )
    
    if (-not (Test-Path $pathToEpic)) {
        Write-Error "Epic WITD not found at $pathToEpic"
        exit 1
    }
    
    if ((Get-Alias -Name witadmin -ErrorAction SilentlyContinue) -eq $null) {
        New-Alias witadmin -Value $pathToWitAdmin
    }
    
    $valueAreadFieldXml = '
    &lt;FIELD name="Value Area" refname="Microsoft.VSTS.Common.ValueArea" type="String"&gt;
        &lt;REQUIRED /&gt;
        &lt;ALLOWEDVALUES&gt;
            &lt;LISTITEM value="Architectural" /&gt;
            &lt;LISTITEM value="Business" /&gt;
        &lt;/ALLOWEDVALUES&gt;
        &lt;DEFAULT from="value" value="Business" /&gt;
        &lt;HELPTEXT&gt;Business = delivers value to a user or another system; Architectural = work to support other stories or components&lt;/HELPTEXT&gt;
    &lt;/FIELD&gt;'
    $valueAreaFieldFormXml = '&lt;Control FieldName="Microsoft.VSTS.Common.ValueArea" Type="FieldControl" Label="Value area" LabelPosition="Left" /&gt;'
    
    $epicCategoryXml = '
    &lt;CATEGORY name="Epic Category" refname="Microsoft.EpicCategory"&gt;
      &lt;DEFAULTWORKITEMTYPE name="Epic" /&gt;
    &lt;/CATEGORY&gt;'
    
    $epicBacklogXml = '
        &lt;PortfolioBacklog category="Microsoft.EpicCategory" pluralName="Epics" singularName="Epic" workItemCountLimit="1000"&gt;
          &lt;States&gt;
            &lt;State value="New" type="Proposed" /&gt;
            &lt;State value="Active" type="InProgress" /&gt;
            &lt;State value="Resolved" type="InProgress" /&gt;
            &lt;State value="Closed" type="Complete" /&gt;
          &lt;/States&gt;
          &lt;Columns&gt;
            &lt;Column refname="System.WorkItemType" width="100" /&gt;
            &lt;Column refname="System.Title" width="400" /&gt;
            &lt;Column refname="System.State" width="100" /&gt;
            &lt;Column refname="Microsoft.VSTS.Scheduling.Effort" width="50" /&gt;
            &lt;Column refname="Microsoft.VSTS.Common.BusinessValue" width="50" /&gt;
            &lt;Column refname="Microsoft.VSTS.Common.ValueArea" width="100" /&gt;
            &lt;Column refname="System.Tags" width="200" /&gt;
          &lt;/Columns&gt;
          &lt;AddPanel&gt;
            &lt;Fields&gt;
              &lt;Field refname="System.Title" /&gt;
            &lt;/Fields&gt;
          &lt;/AddPanel&gt;
        &lt;/PortfolioBacklog&gt;'
    $epicColorXml = '&lt;WorkItemColor primary="FFFF7B00" secondary="FFFFD7B5" name="Epic" /&gt;'
    
    #####################################################################
    function Add-Fragment(
        [System.Xml.XmlNode]$node,
        [string]$xml
    ) {
        $newNode = $node.OwnerDocument.ImportNode(([xml]$xml).DocumentElement, $true)
        [void]$node.AppendChild($newNode)
    }
    
    function Add-ValueAreaField(
        [string]$filePath,
        [string]$controlGroup
    ) {
        $xml = [xml](gc $filePath)
        # check if the field already exists
        if (($valueAreaField = $xml.WITD.WORKITEMTYPE.FIELDS.ChildNodes | ? { $_.refname -eq "Microsoft.VSTS.Common.ValueArea" }) -ne $null) {
            Write-Host "Work item already has Value Area field" -ForegroundColor Yellow
        } else {
            # add field to FIELDS
            Add-Fragment -node $xml.WITD.WORKITEMTYPE.FIELDS -xml $valueAreadFieldXml
    
            # add field to FORM
            # find the "Classification" Group
            $classificationGroup = (Select-Xml -Xml $xml -XPath "//Layout//Group[@Label='$layoutGroupToAddValueAreaControlTo']").Node
            Add-Fragment -node $classificationGroup.Column -xml $valueAreaFieldFormXml
    
            # upload definition
            $xml.Save((gi $filePath).FullName)
            witadmin importwitd /collection:$tpcUri /p:$project /f:$filePath
        }
    }
    #####################################################################
    
    $defaultControlGroup = "Classification"
    switch ($baseTemplate) {
        "Agile" { $wit = "User Story" }
        "Scrum" { $wit = "Product Backlog Item"; $defaultControlGroup = "Details" }
        "CMMI" { $wit = "Requirement" }
    }
    if ($layoutGroupToAddValueAreaControlTo -ne $null) {
        $defaultControlGroup = $layoutGroupToAddValueAreaControlTo
    }
    
    Write-Host "Exporting requirement work item type $wit" -ForegroundColor Cyan
    witadmin exportwitd /collection:$tpcUri /p:$project /n:$wit /f:"RequirementItem.xml"
    
    Write-Host "Adding 'Value Area' field to $wit" -ForegroundColor Cyan
    Add-ValueAreaField -filePath ".\RequirementItem.xml" -controlGroup $defaultControlGroup
    
    Write-Host "Exporting work item type Feature" -ForegroundColor Cyan
    witadmin exportwitd /collection:$tpcUri /p:$project /n:Feature /f:"Feature.xml"
    
    Write-Host "Adding 'Value Area' field to Feature" -ForegroundColor Cyan
    Add-ValueAreaField -filePath ".\Feature.xml" -controlGroup $defaultControlGroup
    
    if (((witadmin listwitd /p:$project /collection:$tpcUri) | ? { $_ -eq "Epic" }).Count -eq 1) {
        Write-Host "Process Template already contains an Epic work item type" -ForegroundColor Yellow
    } else {
        Write-Host "Adding Epic" -ForegroundColor Cyan
        witadmin importwitd /collection:$tpcUri /p:$project /f:$pathToEpic
    }
    
    witadmin exportcategories /collection:$tpcUri /p:$project /f:"categories.xml"
    $catXml = [xml](gc "categories.xml")
    if (($catXml.CATEGORIES.ChildNodes | ? { $_.name -eq "Epic Category" }) -ne $null) {
        Write-Host "Epic category already exists" -ForegroundColor Yellow
    } else {
        Write-Host "Updating categories" -ForegroundColor Cyan
        Add-Fragment -node $catXml.CATEGORIES -xml $epicCategoryXml
        $catXml.Save((gi ".\categories.xml").FullName)
        witadmin importcategories /collection:$tpcUri /p:$project /f:"categories.xml"
    
        Write-Host "Updating ProcessConfig" -ForegroundColor Cyan
        witadmin exportprocessconfig /collection:$tpcUri /p:$project /f:"processConfig.xml"
        $procXml = [xml](gc "processConfig.xml")
    
        Add-Fragment -node $procXml.ProjectProcessConfiguration.PortfolioBacklogs -xml $epicBacklogXml
        Add-Fragment -node $procXml.ProjectProcessConfiguration.WorkItemColors -xml $epicColorXml
    
        $featureCat = $procXml.ProjectProcessConfiguration.PortfolioBacklogs.PortfolioBacklog | ? { $_.category -eq "Microsoft.FeatureCategory" }
        $parentAttrib = $featureCat.OwnerDocument.CreateAttribute("parent")
        $parentAttrib.Value = "Microsoft.EpicCategory"
        $featureCat.Attributes.Append($parentAttrib)
    
        $procXml.Save((gi ".\processConfig.xml").FullName)
        witadmin importprocessconfig /collection:$tpcUri /p:$project /f:"processConfig.xml"
    }
    
    Write-Host "Done!" -ForegroundColor Green

## Running the Script

To run the script, just make sure you’re a Team Project administrator and log in to a machine that has witadmin.exe on it. Then open Team Explorer, connect to your server, and click Settings. Then click “Process Template Manager” and download the new template (Agile, Scrum or CMMI) to a folder somewhere. You really only need the Epic work item WITD. Make a note of where the Epic.xml file ends up.

Then you’re ready to run the script. You’ll need to supply:

- (Optional) The TPC Uri (defaults is http://localhost:8080/tfs/defaultcollection)
- The Team Project name
- The path to the Epic.xml file
- The name of the base template – either Agile, Scrum or CMMI
- (Optional) The path to witadmin.exe (defaults to C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\witadmin.exe)
- (Optional) The name of the group you want to add the “Value Area” field to on the form – default is “Classification”

You can run

<!--kg-card-begin: html--><font size="3" face="Cordia New">Get-Help .\Upgrade-TemplateTo2015.ps1</font><!--kg-card-end: html-->

to get help and examples.

Bear in mind that this script is a “best-effort” – make sure you test it in a test environment before going gung-ho on your production server!

## Results

After running the script. you’ll be able to create Epic work items:

<!--kg-card-begin: html-->[![image](/assets/images/files/76dbf0dc-5910-4ed4-a2d2-f5b39dbe0656.png "image")](/assets/images/files/bb4f7202-9bc1-4f81-a9ea-edf90a783768.png)<!--kg-card-end: html-->

You’ll be able to opt in/out of the Epic backlog in the Team Settings page:

<!--kg-card-begin: html-->[![image](/assets/images/files/8326d637-7260-4617-9c10-c8568f793fb4.png "image")](/assets/images/files/21cc8c7d-d422-4af5-87f0-acfb586e93b5.png)<!--kg-card-end: html-->

You’ll see “Value Area” on your Features and “Requirements”:

<!--kg-card-begin: html-->[![image](/assets/images/files/6f88f50f-4c71-4e68-974a-b359bcf91a10.png "image")](/assets/images/files/b55b60e2-7e74-4e3c-8edb-5dde3793208a.png)<!--kg-card-end: html-->

Happy upgrading!

