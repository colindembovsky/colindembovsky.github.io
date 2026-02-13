---
layout: post
title: Bulk Migrate Work Item Comments, Links and Attachments
date: '2014-08-26 02:50:57'
tags:
- tfsapi
---

I was working at a customer that had set up a test TFS environment. When we set up their “real” TFS, they did a get-latest of their code and imported their code – easy enough. They did have about 100 active work items that they also wanted to migrate. Not being a big fan of TFS Integration Platform, I usually recommend using Excel to port work items _en masse_.

There are a couple of “problems” with the Excel approach:

1. When you create work items in the new Team Project, they have to go into the “New” state (or the first state for the work item)
2. You can’t migrate test cases (since the steps don’t play nicely in Excel) – and you can’t migrate test results either.
3. You can’t migrate comments, hyperlinks or attachments in Excel (other than opening each work item one by one)

You can mitigate the “new state” limitation by creating several sheets – one for “New” items, one for “Active” items, one for “Resolved” items and so on. The “New” items are easy – just import “as-is”. For the other states, import them into the “New” state and then bulk update the state to the “target” state. Keeping the sheets separated by state makes this easier to manage. Another tip I advise is to add a custom field to the new Team Project (you don’t have to expose it on the forms if you don’t want to) called “OldID” that you set to the id of the old work item – that way you’ve always got a link back to the original work item if you need it.

For test case, you have to go to the API to migrate them over to the new team project – I won’t cover that topic in this post.

For comments, hyperlinks and attachments I quickly wrote a PowerShell script that does exactly that! I’ve uploaded it to OneDrive so you can download it [here](http://1drv.ms/1vFDpr5).

Here’s the script itself:

    $oldTpcUrl = "http://localhost:8080/tfs/oldCollection"
    $newTpcUrl = "http://localhost:8080/tfs/newCollection"
    
    $csvFile = ".\map.csv" #format: oldId, newId
    $user = "domain\user"
    $pass = "password"
    
    [Reflection.Assembly]::LoadWithPartialName('Microsoft.TeamFoundation.Common')
    [Reflection.Assembly]::LoadWithPartialName('Microsoft.TeamFoundation.Client')
    [Reflection.Assembly]::LoadWithPartialName('Microsoft.TeamFoundation.WorkItemTracking.Client')
    
    $oldTpc = [Microsoft.TeamFoundation.Client.TfsTeamProjectCollectionFactory]::GetTeamProjectCollection($oldTpcUrl)
    $newTpc = [Microsoft.TeamFoundation.Client.TfsTeamProjectCollectionFactory]::GetTeamProjectCollection($newTpcUrl)
    
    $oldWorkItemStore = $oldTpc.GetService([Microsoft.TeamFoundation.WorkItemTracking.Client.WorkItemStore])
    $newWorkItemStore = $newTpc.GetService([Microsoft.TeamFoundation.WorkItemTracking.Client.WorkItemStore])
    
    $list = Import-Csv $csvFile
    $cred = new-object System.Net.NetworkCredential($user, $pass)
    
    foreach($map in $list) {
        $oldItem = $oldWorkItemStore.GetWorkItem($map.oldId)
        $newItem = $newWorkItemStore.GetWorkItem($map.newId)
    
        Write-Host "Processing $($map.oldId) -&gt; $($map.newId)" -ForegroundColor Cyan
        
        foreach($oldLink in $oldItem.Links | ? { $_.BaseType -eq "HyperLink" }) {
            Write-Host " processing link $($oldLink.Location)" -ForegroundColor Yellow
    
            if (($newItem.Links | ? { $_.Location -eq $oldLink.Location }).count -gt 0) {
                Write-Host " ...link already exists on new work item"
            } else {
                $newLink = New-Object Microsoft.TeamFoundation.WorkItemTracking.Client.Hyperlink -ArgumentList $oldLink.Location
                $newLink.Comment = $oldLink.Comment
                $newItem.Links.Add($newLink)
            }
        }
    
        if ($oldItem.Attachments.Count -gt 0) {
            foreach($oldAttachment in $oldItem.Attachments) {
                mkdir $oldItem.Id | Out-Null
                Write-Host " processing attachment $($oldAttachment.Name)" -ForegroundColor Magenta
    
                if (($newItem.Attachments | ? { $_.Name.Contains($oldAttachment.Name) }).count -gt 0) {
                    Write-Host " ...attachment already exists on new work item"
                } else {
                    $wc = New-Object System.Net.WebClient
                    $file = "$pwd\$($oldItem.Id)\$($oldAttachment.Name)"
    
                    $wc.Credentials = $cred
                    $wc.DownloadFile($oldAttachment.Uri, $file)
    
                    $newAttachment = New-Object Microsoft.TeamFoundation.WorkItemTracking.Client.Attachment -ArgumentList $file, $oldAttachment.Comment
                    $newItem.Attachments.Add($newAttachment)
                }
            }
        
            try {
                $newItem.Save();
                Write-Host " Attachments and links saved" -ForegroundColor DarkGreen
            }
            catch {
                Write-Error "Could not save work item $newId"
                Write-Error $_
            }
        }
    
        $comments = $oldItem.GetActionsHistory() | ? { $_.Description.length -gt 0 } | % { $_.Description }
        if ($comments.Count -gt 0){
            Write-Host " Porting $($comments.Count) comments..." -ForegroundColor Yellow
            foreach($comment in $comments) {
                Write-Host " ...adding comment [$comment]"
                $newItem.History = $comment
                $newItem.Save()
            }
        }
        
        Write-Host "Done!" -ForegroundColor Green
    }
    
    Write-Host
    Write-Host "Migration complete"

When you run this, open the script and fix the top 5 lines (the variables for this script). Enter in the Team Project Collection URL’s (these can be the same if you’re migrating links from one Team Project to another in the same Collection). The person running the script needs to have read permissions on the old server and contributor permission on the new server. You then need to make a cvs file with 2 columns: oldId and newId. Populate this with the mapping from the old work item Id to the new work item Id. Finally, enter a username and password (this is simply for fetching the attachments) and you can run the script.

Happy migrating!

