---
layout: post
title: More on Fakes – the beta has issues
date: '2012-04-04 18:34:00'
tags:
- testing
---

Thanks to [Peter Provost](http://www.peterprovost.org/blog/) for helping answer a couple of questions I had about Fakes – you can look at some of my code in my previous posts about [using Fakes with the TFS API](http://colinsalmcorner.blogspot.com/2012/03/using-fakes-framework-to-test-tfs-api.html).

There were two scenarios that I hit snags with. The first was faking methods in a class that are actually defined on the class’s base class. The second was faking an ObservableCollection.

## Faking Base Class Methods

Consider this code:

    [TestMethod]<br>public void BaseMethodWontWorkInBeta()<br>{<br> using (ShimsContext.Create())<br> {<br> var shimWorkItemStore = new ShimWorkItemStore();<br> var shimTfsTeamProjectCollection = new ShimTfsTeamProjectCollection();<br> var shimTfsConnection = new ShimTfsConnection(shimTfsTeamProjectCollection);<br> shimTfsConnection.GetServiceOf1<workitemstore>(() =&gt; shimWorkItemStore);<br><br> var workItemStore = shimTfsTeamProjectCollection.Instance.GetService<workitemstore>();<br><br> Assert.AreSame(shimWorkItemStore.Instance, workItemStore);<br> }<br>}</workitemstore></workitemstore>

The exercise here is to try to fake the TfsTeamProjectCollection.GetService method. However, we can’t do it directly on the TfsTeamProjectCollection object, since the method is defined in its base class, TfsConnection.

In line 8, we’re using the ShimTfsConnection constructor that takes in an instance of its child class (the TfsTeamProjectCollection) in order to override the method in the child class. This should work, but Peter told me that this doesn’t work in the Beta – it’ll work when the next release of fakes comes out. The workaround is to use the ShimTfsConnection.AllInstances class to override the GetService method (see my earlier posts where I show how to do this).

## Faking ObservableCollection

Let’s look at some more code:

    [TestMethod]<br>public void BindWontWorkBecauseOfObservableCollectionInBeta()<br>{<br> using (ShimsContext.Create())<br> {<br> var ss = new StubISharedStep();<br> var sharedStepReference = new StubISharedStepReference<br> {<br> FindSharedStep = () =&gt; ss<br> };<br><br> var actions = new List<itestaction><br> {<br> sharedStepReference<br> };<br><br> var fakeTestActions = new ShimTestActionCollection();<br> fakeTestActions.Bind((IList<itestaction>)actions);<br><br> Assert.AreEqual(1, fakeTestActions.Instance.Count);<br><br> var ssr = (ISharedStepReference)(fakeTestActions.Instance[0]);<br> Assert.AreSame(sharedStepReference, ssr);<br><br> Assert.AreSame(ss, ssr.FindSharedStep());<br> }<br>}</itestaction></itestaction>

The point of this code is to try to fake a TestActionCollection, which is an ObservableCollection. I had successfully managed to fake other collections that were not ObservableCollections (like a WorkItemCollection), and to do that I used the Bind() method of the Shim to bind the fake collection to a list of items. However, I couldn’t get the code to work on the ObservableCollection. (In the above code, I get a NullReferenceException on line 18 when calling the Bind() method.

Peter confirmed that the reason for this is twofold: one, I hadn’t added Fakes for System, which is where the ObservableCollection class resides. This is a performance optimization that the Fakes creation employs so that they don’t create fakes for classes that you’ll never use.

Secondly, and rather unfortunately, even adding that Fake doesn’t work in the Beta. The Fakes creation uses a white-list to generate fakes in the System namespace, and the ObservableCollection isn’t on the white-list in the Beta. In the next release, the System white-list will be customizable, so I am looking forward to getting my grubby paws on that!

Unfortunately, this scenario has no work-around in the Beta, so if you’re trying to Fake an ObservableCollection, you’ll have to wait for the next release of Fakes.

I really like the Fakes framework and what it can do for testing, so I am looking forward to seeing what else comes out of the woodwork.

Happy faking!

