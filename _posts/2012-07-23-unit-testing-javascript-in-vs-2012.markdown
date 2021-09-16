---
layout: post
title: Unit Testing Javascript in VS 2012
date: '2012-07-23 19:24:00'
tags:
- development
---

You’re a responsible developer – you write code, and then you write tests (or, perhaps you even write tests and then write code?). You also love good, solid frameworks that separate concerns and utilize dependency injection and inversion of control and all of that good stuff – you’re using MVC for your web applications.

But you have a little problem – your pages use Javascript, and that’s difficult to unit test, right? Not any more.

## The Sample Solution

For this post I’ll use a sample MVC app – of course, the principles apply to any client side app using Javascript, but this is an easy example to use.

Create a new MVC project in VS, and then add [this script](http://sdrv.ms/NO71vH) to the Scripts folder. This script contains some Javascript for populating a country / state dropdown combo. This is the script that we’re going to be testing.

## Chutzpah and QUnit for MVC via NuGet

You’re going to need a Javascript testing framework, and then the ability to run your tests. There are a few frameworks out there, but one I enjoy using is [QUnit](http://docs.jquery.com/QUnit). But that only gives you the infrastructure – you want to be able to run the tests easily too. Fortunately, VS 2012’s Test Explorer is extensible, so you can add your own “test adapters”. And there’s already one by Matthew Manela for Javascript testing with QUnit called Chutzpah (there is a [test runner](http://chutzpah.codeplex.com/) and a [Test Explorer extension](http://visualstudiogallery.msdn.microsoft.com/f8741f04-bae4-4900-81c7-7c9bfb9ed1fe) – we’ll just need the extension for now).

Let’s create a unit test project and use [NuGet](http://nuget.org/) to install the QUnit package.

1. Create a new C# Unit Test project. Delete the UnitTest1.cs file.
2. Open the Package Manager Console (NuGet) and make sure the test project is set to the default project.
3. Type “Install-Package QUnit-MVC” to install QUnit for MVC (this can be used to test any javascript, not just MVC).
4. In the Extension Manager, search for Chutzpah and install the Test Adapter extension. Remember to restart VS once you’ve installed the extension.
<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-pts2hE6n4vk/UA0mObZsnHI/AAAAAAAAAbY/HE6NwM75sfE/image_thumb%25255B2%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-W90N1TOLu-w/UA0mNfQyJFI/AAAAAAAAAbQ/zSOiUXw6SNA/s1600-h/image%25255B6%25255D.png)<!--kg-card-end: html-->
## Writing Tests

Let’s create a test stub for our unit test. Create a new Javascript file in the test project called “StateDropDownTests.js”. Put in the following code:

    /// <reference path="../MvcApplication/Scripts/jQuery-1.6.2.js"><br>/// <reference path="../MvcApplication/Scripts/StateDropDown.js"><br><br><br>module("State Dropdown Tests:")<br><br>test("test States", function () {<br><br>});<br></reference></reference>

Now open the Test Explorer and click “Run All”. You should see the test come up in the list of tests and it should fail (since nothing was tested).

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-e07IlfMvivc/UA0mSBLIVsI/AAAAAAAAAbo/AaE3F7MLOic/image_thumb%25255B4%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-hvsCDS8D2SA/UA0mPvQ-MKI/AAAAAAAAAbg/80TVW4D7GVw/s1600-h/image%25255B10%25255D.png)<!--kg-card-end: html-->

## The Real Test Code

Since the StateDropdown script we’re testing accesses objects from the DOM, you’re going to need to add some basic elements. Fortunately QUnit has hooks for adding elements into a _test fixture_. Add the following code to the module to create the DOM element we need for testing:

    module("State Dropdown Tests:", {<br> setup: function () {<br> $("#qunit-fixture").append(<br> '<select id="countrySelect" name="country">' + '<option value="USA">USA</option>' + '</select>' +<br> '<select id="stateSelect" name="state">' + '</select>'<br> );<br> }<br>});<br>

Now add the following code into your test:

    test("test States", function () {<br> postCountry = '';<br> postState = '';<br><br> var stateSelect = document.getElementById('stateSelect');<br><br> // the dropdown includes all the states PLUS a 'Select State' option<br> initCountry('US');<br> equal(stateSelect.options.length, 62 + 1);<br><br> initCountry('UK');<br> equal(stateSelect.options.length, 43 + 1);<br><br> initCountry('CA');<br> equal(stateSelect.options.length, 15 + 1);<br><br> initCountry('ZA');<br> equal(stateSelect.options.length, 0 + 1);<br>});<br>

Run the test – and you should see lots of green! Our test is working.

I was going to write a follow on post about getting the tests to run in TeamBuild, but Mathew Aniyan from the MS ALM team beat me to it – here’s a link to his post about

<!--kg-card-begin: html--><font style="font-weight: normal"><a href="http://blogs.msdn.com/b/visualstudioalm/archive/2012/07/09/javascript-unit-tests-on-team-foundation-service-with-chutzpah.aspx" target="_blank">Javascript Unit Tests on Team Foundation Service with Chutzpah</a></font><!--kg-card-end: html-->

Happy Javascript testing!

