---
layout: post
title: Microsoft Fakes – Customizing the System Whitelist (or, enabling Fakes for
  classes you’ve always wanted to fake, like WebClient)
date: '2012-06-25 20:35:00'
tags:
- testing
---

So you’re sitting down planning some tests for your shiny new code, only to find that your code uses WebClient to download a file. No problem – you’ve been reading about Microsoft’s new Fakes framework, so you just right-click the System reference in your test project and select “Create Fakes” and you get a bunch of cool fakes to work with.

But going a bit deeper into the Rabbit Hole, you realize that there are no fakes for System.Net classes. What gives?

## The White-List

It turns out that when you right-click a reference and select “Add Fakes” a fakes file is created for that assembly in the Fakes folder. When you add a Fakes lib for System, you in fact get 2 fakes files: one for mscorlib and one for System.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-3N5Oqrx3-6s/T-hM93wWrLI/AAAAAAAAAZM/aEt9u8h-zbg/image_thumb.png?imgmax=800 "image")](http://lh5.ggpht.com/-7RSmvbea-NI/T-hM8-M9AyI/AAAAAAAAAZE/RX-xVTCfsFQ/s1600-h/image2.png)<!--kg-card-end: html-->

Since System is a large library, the Fakes framework doesn’t automatically generate you a fake for every System class. One of the classes that doesn’t have a fake created by default is System.Net.WebClient. In order to force a fake Shim for this class, you’ll need to override the default “white-list” of classes. Fortunately this is relatively straight-forward to do.

## The Fakes Config File

Double-click System.fakes to open it in the editor – it’s just an XML file. It does come with an XSD, so as you type in this file IntelliSense will guide you (just like the Force, Luke). Within the &nbsp;tag, add a &nbsp;tag and within that an &nbsp;tag and you’re good to go. Here’s my Fakes file for adding a WebClient fake:

    <fakes xmlns="http://schemas.microsoft.com/fakes/2011/"><br> <assembly name="System" version="4.0.0.0"><br> <shimgeneration><br> <add fullname="System.Net.WebClient"><br> </add></shimgeneration><br></assembly></fakes><br>

Remember to recompile!

Here’s some example code using the WebClient fake:

    public class HttpFunctions<br>{<br> public void Download(string url, string fileName)<br> {<br> var w = new WebClient();<br> w.DownloadFile(url, fileName);<br> <br> }<br>}<br><br>[TestClass]<br>public class UnitTest1<br>{<br> [TestMethod]<br> public void TestFakeWebClient()<br> {<br> using (ShimsContext.Create())<br> {<br> ShimWebClient.AllInstances.DownloadFileStringString = (w, url, fileName) =&gt;<br> {<br> File.WriteAllText(fileName, url);<br> };<br><br> var c = new HttpFunctions();<br> string guidFileName = Guid.NewGuid().ToString() + ".html";<br> c.Download("http://www.bing.com", guidFileName);<br><br> Assert.IsTrue(File.Exists(guidFileName));<br> string contents = File.ReadAllText(guidFileName);<br> Assert.AreEqual("http://www.bing.com", contents);<br> File.Delete(guidFileName);<br> }<br> }<br>}<br>

Happy faking!

