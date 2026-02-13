---
layout: post
title: Using the Fakes Framework to Test TFS API Code (Part 1 of 2)
date: '2012-03-20 20:20:00'
tags:
- tfsapi
- testing
---

(Here’s the link to [Part 2](http://colinsalmcorner.blogspot.com/2012/03/using-fakes-framework-to-test-tfs-api_20.html))

If you’ve ever written a utility to do TFS “stuff” using the TFS API, you probably tested by hitting F5 and stepping through a bit before letting any large loops do their thing. So what happened to all the goodness is expected of good developers – for example unit testing?

Well, it turns out it’s insanely difficult to test any code that depends on the TFS APIs. You could craft tests that hit an actual TFS server, but then you’d have to worry about clean up so that your tests could be repeated. And of course you’d \*never\* do this sort of testing against a Live server (or a Live project), right?

Besides, unit tests, at least in theory, are supposed to have no dependencies on external systems. So how do you go about unit testing code that uses the TFS API?

## Scenario: Copy Work Items

Let’s imagine you’ve written a console app to copy work items (obtained from executing a stored query) to a target iteration. Here’s the code for the WorkItemCopyer class and Program.cs:

    class WorkItemCopyer<br>{<br> public WorkItemCollection WorkItems { get; private set; }<br> public QueryHierarchy QueryHierarchy { get; private set; }<br><br> public WorkItemStore Store { get; private set; }<br> public TfsTeamProjectCollection TPC { get; private set; }<br> public string TeamProjectName { get; private set; }<br><br> public WorkItemCopyer(TfsTeamProjectCollection tpc, string teamProjectName)<br> {<br> TPC = tpc;<br> TeamProjectName = teamProjectName;<br> Store = TPC.GetService<workitemstore>();<br> QueryHierarchy = Store.Projects[TeamProjectName].QueryHierarchy;<br> }<br><br> public void RunQuery(QueryDefinition queryDef)<br> {<br> var dict = new Dictionary<string, string="">() <br> {<br> { "project", TeamProjectName }, <br> { "me", GetCurrentUserDisplayName() } <br> };<br><br> var query = new Query(Store, queryDef.QueryText, dict);<br> WorkItems = query.RunQuery();<br> }<br><br> private string GetCurrentUserDisplayName()<br> {<br> var securityService = TPC.GetService<igroupsecurityservice>();<br> var accountName = string.Format("{0}\\{1}", Environment.UserDomainName, Environment.UserName);<br> var memberInfo = securityService.ReadIdentity(SearchFactor.AccountName, accountName, QueryMembership.None);<br> if (memberInfo != null)<br> {<br> return memberInfo.DisplayName;<br> }<br> return Environment.UserName;<br> }<br><br> public int CopyWorkItems(string targetIterationPath)<br> {<br> foreach (WorkItem workItem in WorkItems)<br> {<br> var copy = workItem.Copy();<br> copy.IterationPath = targetIterationPath;<br> copy.Save();<br> }<br> return WorkItems.Count;<br> }<br><br> public QueryDefinition FindQuery(string queryName)<br> {<br> return FindQueryInFolder(QueryHierarchy, queryName);<br> }<br><br> private QueryDefinition FindQueryInFolder(QueryFolder folder, string queryName)<br> {<br> foreach (var query in folder.OfType<querydefinition>())<br> {<br> if (query.Name == queryName)<br> {<br> return query;<br> }<br> }<br> QueryDefinition subQuery = null;<br> foreach (var subFolder in folder.OfType<queryfolder>())<br> {<br> subQuery = FindQueryInFolder(subFolder, queryName);<br> if (subQuery != null)<br> {<br> return subQuery;<br> }<br> }<br> return null;<br> }<br>}<br><br>class Program<br>{<br> static void Main(string[] args)<br> {<br> var tpcUrl = args[0];<br> var teamProjectName = args[1];<br> var queryName = args[2];<br> var targetIterationPath = args[3];<br><br> var tpc = TfsTeamProjectCollectionFactory.GetTeamProjectCollection(new Uri(tpcUrl));<br> var copyer = new WorkItemCopyer(tpc, teamProjectName);<br><br> var query = copyer.FindQuery(queryName);<br> copyer.RunQuery(query);<br> var count = copyer.CopyWorkItems(targetIterationPath);<br><br> Console.WriteLine("Successfully copied {0} work items", count);<br> Console.WriteLine("Press <enter> to quit...");<br> Console.ReadLine();<br> }<br>}</enter></queryfolder></querydefinition></igroupsecurityservice></string,></workitemstore>

Now we get to the interesting part: unit testing. Let’s start off assuming we have a test project that we can run the tests against. A CopyTest would look something like this:

    [TestMethod]<br>public void TestCopyWithDependencies()<br>{<br> var tpc = TfsTeamProjectCollectionFactory.GetTeamProjectCollection(new Uri("http://localhost:8080/tfs/defaultcollection"));<br> var target = new WorkItemCopyer(tpc, "Code11");<br><br> // test finding the query<br> var query = target.FindQuery("Tasks_Release1_Sprint1");<br> Assert.IsNotNull(query);<br><br> // test running the query<br> target.RunQuery(query);<br> Assert.IsTrue(target.WorkItems.Count &gt; 0);<br><br> // test copy<br> var count = target.CopyWorkItems(@"Code11\Release 1\Sprint 2");<br> Assert.AreEqual(target.WorkItems.Count, count);<br>}

But there’s a problem here: if we run these tests and the server is down, they’ll fail. If someone renames the query, the tests will fail. If for some reason the query return 0 work items, the test will at best be inconclusive. So we clearly need to isolate the test from the TFS server. Enter the [Fakes framework](http://msdn.microsoft.com/en-us/library/hh549175(v=vs.110).aspx).

## Fakes

The [Fakes Framework](http://msdn.microsoft.com/en-us/library/hh549175(v=vs.110).aspx) came out of the [Moles Framework](http://research.microsoft.com/en-us/projects/moles/) from the MS RiSE team. It allows you to isolate your test code from almost anything using an interception mechanism.

To use Fakes, you’ll first need to create the Fake assemblies. We’ll right click each TeamFoundation dll reference, select “Create Fakes” and we’ll be ready to go.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-HucNptcHp-I/T2hn2CoGRwI/AAAAAAAAAX4/oMDXUB5rlWs/image_thumb.png?imgmax=800 "image")](http://lh3.ggpht.com/-1trIVFT6m6k/T2hn0rjMbrI/AAAAAAAAAXw/YFaKRhncgUg/s1600-h/image2.png)<!--kg-card-end: html-->

You’ll see the Fakes assemblies in your References now.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-SS289Mihruo/T2hn5St6rTI/AAAAAAAAAYI/WT_WuO-u7ko/image_thumb1.png?imgmax=800 "image")](http://lh3.ggpht.com/-xuiTHWjdifI/T2hn3qSc-qI/AAAAAAAAAYA/nZONPDWxEb4/s1600-h/image5.png)<!--kg-card-end: html-->

## Faking enough to Instantiate a WorkItemCopyer

The first thing you need to know about fakes is that they only work inside a ShimsContext, which you can wrap into a using. In order to construct the WorkItemCopyer, we’re going to need a TeamProjectCollection object. So let’s see if we can fake it:

    [TestMethod]<br>public void TestCopyerInstantiate()<br>{<br> using (ShimsContext.Create())<br> {<br> // set up the fakes<br> var fakeTPC = new ShimTfsTeamProjectCollection();<br> <br> var target = new WorkItemCopyer(fakeTPC, "Code11");<br> Assert.IsNotNull(target);<br> }<br>}

If you run this code, you’ll get an error in the WorkItemCopyer constructor:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-jLModFC2D_s/T2hn9wnpXzI/AAAAAAAAAYY/Y652U7eEqfg/image_thumb3.png?imgmax=800 "image")](http://lh6.ggpht.com/-5EAFR3qa6QU/T2hn8F4l97I/AAAAAAAAAYQ/B-tg57HeJKI/s1600-h/image9.png)<!--kg-card-end: html-->

The GetService method seems to be confused. We’ll need to fake that call to return a fake WorkItemStore. So we create a fake store (newing up a ShimWorkItemStore). But now how do we fix the GetService call? You’ll notice it’s fake counterpart is not on the ShimTeamProjectCollection class. This is because the method doesn’t exist on the TeamProjectCollection class, but on it’s base class, TfsConnection. Now according to the MSDN Fakes documentation (which talks about faking methods in base classes), I would have expected this code to work:

    var fakeStore = new ShimWorkItemStore();<br>var fakeTPC = new ShimTfsTeamProjectCollection();<br>var fakeBase = new ShimTfsConnection(fakeTPC);<br>fakeBase.GetServiceOf1<workitemstore>(() =&gt; fakeStore);</workitemstore>

But for some reason, this doesn’t work. So we’ll do the next best thing is to fake the GetService method for all instances of TfsConnection (which will include and TeamProjectCollection object as well). Here’s the code now:

    [TestMethod]<br>public void TestCopyerInstantiate()<br>{<br> using (ShimsContext.Create())<br> {<br> // set up the fakes<br> var fakeStore = new ShimWorkItemStore();<br> var fakeTPC = new ShimTfsTeamProjectCollection();<br> ShimTfsConnection.AllInstances.GetServiceOf1<workitemstore>((t) =&gt; fakeStore);<br> <br> var target = new WorkItemCopyer(fakeTPC, "Code11");<br> Assert.IsNotNull(target);<br> }<br>}</workitemstore>

Now we get a bit further – we get to the code that initializes the QueryHierarchy property and then we get a ShimNotImplementedException:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-FD549AZrBs8/T2hoCOPaafI/AAAAAAAAAYo/PP4wDZWstgw/image_thumb5.png?imgmax=800 "image")](http://lh6.ggpht.com/-21NO1X4TrZA/T2hoADyBOFI/AAAAAAAAAYg/Ix6oy94VwIA/s1600-h/image13.png)<!--kg-card-end: html-->

This is because the getter method Projects on our fake store is not faked. So we’ll change the code to return a fake TeamProjectCollection, which in turn needs a fake string indexer method to get a TeamProject which in turn needs a fake QueryHierarchy getter method… to save time, I’ll show you the completed code:

    [TestMethod]<br>public void TestCopyerInstantiate()<br>{<br> using (ShimsContext.Create())<br> {<br> // set up the fakes<br> var fakeHierarchy = new ShimQueryHierarchy();<br> var fakeProject = new ShimProject()<br> {<br> NameGet = () =&gt; "TestProject",<br> QueryHierarchyGet = () =&gt; fakeHierarchy<br> };<br> var fakeProjectCollection = new ShimProjectCollection()<br> {<br> ItemGetString = (projectName) =&gt; fakeProject<br> };<br> var fakeStore = new ShimWorkItemStore()<br> {<br> ProjectsGet = () =&gt; fakeProjectCollection<br> };<br> var fakeTPC = new ShimTfsTeamProjectCollection();<br> ShimTfsConnection.AllInstances.GetServiceOf1<workitemstore>((t) =&gt; fakeStore); <br><br> // test<br> var target = new WorkItemCopyer(fakeTPC, "Code11");<br> Assert.IsNotNull(target);<br> }<br>}</workitemstore>

So far so good: we can at least instantiate the WorkItemCopyer. In the [Part 2](http://colinsalmcorner.blogspot.com/2012/03/using-fakes-framework-to-test-tfs-api_20.html) post we’ll fake some Query folders and Query objects as well as some WorkItems so that we can complete a test for copying work items.

Happy faking!

