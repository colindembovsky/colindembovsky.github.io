---
layout: post
title: Using the TFS API to display results of a Hierarchical Work Item Query
date: '2011-10-10 18:37:00'
tags:
- tfsapi
---

## RunQuery won’t work for Hierarchical Queries

Using the TFS API to display results of a flat query is fairly straightforward – once you have the [WIQL](http://msdn.microsoft.com/en-us/library/bb130306.aspx) you just execute the [RunQuery()](http://msdn.microsoft.com/en-us/library/microsoft.teamfoundation.workitemtracking.client.query.runquery.aspx) method and voila – a nice WorkItemCollection for you to enumerate over. However, if you try to execute RunQuery() on a tree or one-hop WIQL, you’ll see this error message:

    TF248021: You have specified a query string that is not valid when you use the query method for a flat list of work items. You cannot specify a parameterized query or a query string for linked work items with the query method you specified.

I was working on a proof-of-concept and needed to display the result of a Work Item Query in a WPF form. Once I saw the TF248012 error, I googled binged a bit to see if I could work out how to run a hierarchical query. That brought me to the [RunLinkQuery()](http://msdn.microsoft.com/en-us/library/microsoft.teamfoundation.workitemtracking.client.query.runlinkquery.aspx) method, which returns a WorkItemLinkInfo array. But that’s where my [google-fu](http://en.wiktionary.org/wiki/Google-fu) ran out of steam – there didn’t seem to be much info about how to proceed once you’ve got this array. So here I’ll show you how I used this array to enumerate the work item hierarchy.

I’ve uploaded the code to my [skydrive](https://skydrive.live.com/?cid=64a24e0938d6d062&sc=documents&uc=1&id=64A24E0938D6D062%21303#) if you want to download it. (Note: If you want to copy code from the snippets below, just double click in the code area and Cntrl-C).

## Phase 1 – Setting up the QueryRunner Class

In order to work with hierarchical results, you need to execute the query first! Here’s how I did it:

    class QueryRunner{ public WorkItemStore WorkItemStore { get; private set; } public string TeamProjectName { get; private set; } public string CurrentUserDisplayName { get; private set; } public QueryRunner(string tpcUrl, string teamProjectName) { var tpc = TfsTeamProjectCollectionFactory.GetTeamProjectCollection(new Uri(tpcUrl)); WorkItemStore = tpc.GetService<workitemstore>(); TeamProjectName = teamProjectName; }}</workitemstore>

I defined a class called QueryRunner that has a couple of properties:

- WorkItemStore – the WorkItemStore service
- TeamProjectName – the name of the team project that contains the stored query we want to run
- CurrentUserDisplayName – the display name of the current user (more on this later)

You’ll need references to Microsoft.TeamFoundation, Microsoft.TeamFoundation.Client and Microsoft.TeamFoundation.WorkItemTracking.Client, all of which can be found in the .NET tab of the Add References dialogue.

The constructor takes in the Url of the Team Project Collection and the Team Project Name. It then initializes the WorkItemStore.

Two supporting methods you’ll need are the following:

    private void ResolveCurrentUserDisplayName(){ var securityService = WorkItemStore.TeamProjectCollection.GetService<igroupsecurityservice>(); var accountName = string.Format("{0}\\{1}", Environment.UserDomainName, Environment.UserName); var memberInfo = securityService.ReadIdentity(SearchFactor.AccountName, accountName, QueryMembership.None); if (memberInfo != null) { CurrentUserDisplayName = memberInfo.DisplayName; } else { CurrentUserDisplayName = Environment.UserName; }}private IDictionary GetParamsDictionary(){ return new Dictionary<string string="" ,="">() { { "project", TeamProjectName }, { "me", CurrentUserDisplayName } };}</string></igroupsecurityservice>

ResolveCurrentUserDisplayName uses the IGroupSecurityService to resolve the current user’s display name. This is needed if your WIQL contains the “@Me” macro. You can see how it’s used in the GetParamsDictionary method.

## Phase 2 – Define a Hierarchical Data Structure

Now we’re almost ready to execute queries – we just need a data structure to hold the results. Here’s what I ended up defining:

    class PropertyChangingBase : INotifyPropertyChanged{ public event PropertyChangedEventHandler PropertyChanged; protected void OnPropertyChanged(string propertyName) { if (PropertyChanged != null) { PropertyChanged(this, new PropertyChangedEventArgs(propertyName)); } }}class WorkItemNode : PropertyChangingBase{ private WorkItem workItem; public WorkItem WorkItem { get { return workItem; } set { workItem = value; OnPropertyChanged("WorkItem"); } } private string relationshipToParent; public string RelationshipToParent { get { return relationshipToParent; } set { relationshipToParent = value; OnPropertyChanged("RelationshipToParent"); } } private List<workitemnode> children; public List<workitemnode> Children { get { return children; } set { children = value; OnPropertyChanged("Children"); } }}</workitemnode></workitemnode>

## Phase 3 – Run the Query

Now we’re ready to run the query.

    public List<workitemnode> RunQuery(Guid queryGuid){ // get the query var queryDef = WorkItemStore.GetQueryDefinition(queryGuid); var query = new Query(WorkItemStore, queryDef.QueryText, GetParamsDictionary()); // get the link types var linkTypes = new List<workitemlinktype>(WorkItemStore.WorkItemLinkTypes); // run the query var list = new List<workitemnode>(); if (queryDef.QueryType == QueryType.List) { foreach (WorkItem wi in query.RunQuery()) { list.Add(new WorkItemNode() { WorkItem = wi, RelationshipToParent = "" }); } } else { var workItemLinks = query.RunLinkQuery().ToList(); list = WalkLinks(workItemLinks, linkTypes, null); } return list;}</workitemnode></workitemlinktype></workitemnode>

In this method, we take in the Guid of the query we want to run (how to get that Guid is another discussion – you can see the Guids of stored queries by looking at the TeamProject.QueryHierarchy property). Also, this method is using the WIQL from the stored query, but you could just as well pass in raw WIQL too.

We construct a Query object passing in the WorkItemStore, the WIQL and the parameter dictionary for “@Project” and “@Me” macros. Next we get a list of all the WorkItemLinkTypes in the WorkItemStore. We’ll use these when we enumerate the work item links.

Finally, we decide on whether or not to run a flat query or a hierarchical query based on the stored query type. If you’re passing in WIQL instead, you’ll have to decide some other way. For flat queries, just construct a list of nodes. For hierarchical queries, get the WorkItemLinkInfo array and walk it using the following (recursive) method:

    private List<workitemnode> WalkLinks(List<workitemlinkinfo> workItemLinks, List<workitemlinktype> linkTypes, WorkItemNode current){ var currentId = 0; if (current != null) { currentId = current.WorkItem.Id; } var workItems = (from linkInfo in workItemLinks where linkInfo.SourceId == currentId select new WorkItemNode() { WorkItem = WorkItemStore.GetWorkItem(linkInfo.TargetId), RelationshipToParent = linkInfo.LinkTypeId == 0 ? "Parent" : linkTypes.Single(lt =&gt; lt.ForwardEnd.Id == linkInfo.LinkTypeId).ForwardEnd.Name }).ToList(); workItems.ForEach(w =&gt; w.Children = WalkLinks(workItemLinks, linkTypes, w)); return workItems;}</workitemlinktype></workitemlinkinfo></workitemnode>

This method walks the array, starting with links that have a source ID of 0 (these are top level work items in the hierarchy). For each of those work items, create a WorkItemNode and then populate the children using the current node as the “current” for the next level of recursion. When we go to the next level, the name of the WorkItemLinkType to the parent can be found using the LinkTypeId property of the WorkItemLinkInfo and finding the corresponding ForwardEnd Id in the list of WorkItemLinkTypes.

## Epilogue: Displaying the Result in WPF

Once you’ve got the tree of WorkItemNodes, displaying them in WPF is really easy. Here’s the XAML for the TreeView:

    <treeview horizontalalignment="Stretch" verticalalignment="Stretch" itemssource="{Binding}"> <treeview.itemtemplate> <hierarchicaldatatemplate itemssource="{Binding Children}"> <stackpanel orientation="Horizontal"> <textblock text="{Binding WorkItem.Id, StringFormat=F0}" fontweight="Bold"> <textblock text="{Binding WorkItem.Title}" padding="3, 0, 0, 0"> <textblock text="{Binding RelationshipToParent, StringFormat=({0})}" padding="3, 0, 0, 0" fontstyle="Italic"> </textblock></textblock></textblock></stackpanel> </hierarchicaldatatemplate> </treeview.itemtemplate></treeview>

We specify that the ItemTemplate for the TreeView contains hierarchical data. For each node in the tree, display the Id, Title and RelationshipToParent. Then use the “Children” property to display the next level in the hierarchy. In the code-behind for this XAML, we simply set the DataContext to the QueryRunner.RunQuery() results. Here are some screenshots of the results for a one-hop and a tree query respectively.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-k1VQK6W6Dfw/TpK82FdMHhI/AAAAAAAAATw/z3N1sZDsNB4/image_thumb.png?imgmax=800 "image")](http://lh4.ggpht.com/-NON-EatslBg/TpK81AOND6I/AAAAAAAAATs/uCfNtwhpzgE/s1600-h/image2.png)<!--kg-card-end: html--><!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-XZfHQDCLAhI/TpK830-TpTI/AAAAAAAAAT4/9x4iAudJGtU/image_thumb1.png?imgmax=800 "image")](http://lh3.ggpht.com/-peaYX_PDyUk/TpK827D1pNI/AAAAAAAAAT0/YXwuu9kZObo/s1600-h/image5.png)<!--kg-card-end: html-->

Happy querying!

