---
layout: post
title: Using the Fakes Framework to Test TFS API Code (Part 2 of 2)
date: '2012-03-20 20:22:00'
tags:
- tfsapi
- testing
---

In [Part 1](http://colinsalmcorner.blogspot.com/2012/03/using-fakes-framework-to-test-tfs-api.html), we started faking some TFS objects. We got as far as faking the TeamProjectCollection and WorkItemStore. In this post, we’ll complete the test for copying work items by providing a fake QueryHierarchy and a fake list of WorkItems.

## Binding IEnumerables

Since the QueryHierarchy is an IEnumerable, we’ll need to either fake the GetEnumerator() method, or find a way of making the fake QueryHierarchy enumerable! The problem with trying to fake the GetEnumerator() method is that you end up in an infinite loop – if you’re trying to get the enumerator for sub nodes of the hierarchy, you’ll also need to call GetEnumerator() which will call GetEnumerator() and so on and so on… so we’ll see if we can make the fake QueryHiearchy behave like an enumerable object. Which is really quite easy when you know how!

We create a list of QueryFolders (outside the ShimsContext, since we want these object to be real and not fake) and then call the Bind() method on the fake QueryHierarchy:

    var myQueries = new QueryFolder("My Queries");<br>var sharedQueries = new QueryFolder("Team Queries");<br>sharedQueries.Add(new QueryDefinition("My Tasks", "SELECT System.Id FROM WorkItems WHERE System.WorkItemType = 'Task'"));<br>sharedQueries.Add(new QueryDefinition("My Bugs", "SELECT System.Id FROM WorkItems WHERE System.WorkItemType = 'Bug'"));<br>var topQueries = new List<queryitem>() { myQueries, sharedQueries };<br><br>using (ShimsContext.Create())<br>{<br> // set up the fakes<br> var fakeHierarchy = new ShimQueryHierarchy();<br> fakeHierarchy.Bind(topQueries);<br></queryitem>

Now when we run the code, the QueryHierarchy behaves like an enumeration and a call to find the “My Tasks” query will result in a QueryDefinition being returned!

## Faking an Interface

The next thing we’ll need to fake is the RunQuery() method on the Query. The first snag we’ll hit is that the RunQuery() method in the WorkItemCopyer uses the IGroupSecurityService to resolve the current user’s display name for any @me macros in the query. So we’ll first need to fake that. Hang on – how do you fake an interface?? Enter Stubs.

So far we’ve only used Shims. For faking interfaces, we’ll need to switch to Stubs. The only method on the IGroupSecurityService interface that we want to fake is the ReadIdentity() method. So we can fake that and return a fake Identity object. Again, we’ll run into the problem of wanting a real object in a fake world (i.e. the ShimsContext) so we’ll use a sneaky way of working around that. Here’s the code for the fake IGroupSecurityService:

    var fakeSecurityService = new StubIGroupSecurityService()<br>{<br> ReadIdentitySearchFactorStringQueryMembership = (searchFactor, criteria, identity) =&gt; ShimsContext.ExecuteWithoutShims<identity>(() =&gt; new Identity() { DisplayName = "Bob" })<br>};<br>ShimTfsConnection.AllInstances.GetServiceOf1<igroupsecurityservice>((t) =&gt; fakeSecurityService);<br></igroupsecurityservice></identity>

This will create the Identity object “outside” the current ShimsContext. Of course, we need to tell the TfsConnection GetService call to return the fake security service when asked for an IGroupSecurityService (which is done in the last line above).

## Faking Constructors

The next thing that the WorkItemCopyer code does is instantiate a Query object in order to execute the Work Item Query. We’ll need to fake the constructor in order to return a fake Query. The method we particularly want to fake out on the Query object is the RunQuery() method, which is going to return a WorkItemCollection. We’ll want to create a list of Work Items and bind a fake WorkItemCollection to the list so that we can enumerate the fake results. There is also a call to the IterationPath setter and a couple of getters on the WorkItem itself, so we’ll fake that at the same time. Here’s the code:

    ShimQuery.ConstructorWorkItemStoreStringIDictionary = (q, store, text, dict) =&gt;<br>{<br> new ShimQuery(q)<br> {<br> RunQuery = () =&gt;<br> {<br> var workItemType = new ShimWorkItemType()<br> {<br> NameGet = () =&gt; "Task"<br> };<br> var list = new List<workitem>()<br> {<br> new ShimWorkItem()<br> {<br> IdGet = () =&gt; 12,<br> TitleGet = () =&gt; "Some Work Item",<br> TypeGet = () =&gt; workItemType,<br> IterationPathSetString = (path) =&gt; { },<br> }<br> };<br> var workItems = new ShimWorkItemCollection()<br> {<br> CountGet = () =&gt; list.Count<br> };<br> workItems.Bind(list);<br> return workItems;<br> }<br> };<br>};<br></workitem>

Finally, we’ll create a couple of counters to make sure that our code calls the copy and the save methods of the work items for each work item.

    var copyCount = 0;<br>ShimWorkItem.AllInstances.Copy = (w) =&gt;<br>{<br> copyCount++;<br> return new ShimWorkItem(w);<br>};<br>var saveCount = 0;<br>ShimWorkItem.AllInstances.Save = (_) =&gt; saveCount++;<br><br>// test instantiate<br>var target = new WorkItemCopyer(fakeTPC, "Code11");<br>Assert.IsNotNull(target);<br><br>// test find query<br>var query = target.FindQuery("My Tasks");<br>Assert.IsNotNull(query);<br><br>// test run query<br>target.RunQuery(query);<br>Assert.AreEqual(1, target.WorkItems.Count);<br><br>// test copy work items<br>var count = target.CopyWorkItems("Test Iteration");<br>Assert.AreEqual(1, count);<br>Assert.AreEqual(1, copyCount);<br>Assert.AreEqual(1, saveCount);<br>

Voila! We’ve been able to test our code without actually connecting to a real TFS service. And coverage? Well, it’s up to 97% for the WorkItemCopyer class. Not too bad!

The completed solution is available on my [skydrive](https://skydrive.live.com/redir.aspx?cid=64a24e0938d6d062&resid=64A24E0938D6D062!335&parid=64A24E0938D6D062!334).

(More) Happy Faking!

