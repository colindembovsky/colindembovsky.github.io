---
layout: post
title: 'CrossBrowser Testing: ChromeDriver Window Hangs after Test'
date: '2014-01-31 22:07:00'
tags:
- testing
---

I have been doing some coded UI testing and running tests using Chrome (via the [Selenium components](http://visualstudiogallery.msdn.microsoft.com/11cfc881-f8c9-4f96-b303-a2780156628d)). However, I noticed that when my test completed successfully, the Selenium (ChromeDriver) window stayed open and never terminated. Here’s a code snippet of my original code:

    [TestMethod]
    public void TestTimesheetIsDeployedChrome()
    {
        BrowserWindow.CurrentBrowser = "chrome";
    
        var testUrl = ConfigurationManager.AppSettings["TimesheetUrl"];
        Assert.IsFalse(string.IsNullOrEmpty(testUrl), "Could not find testUrl in App Settings");
        var window = BrowserWindow.Launch(testUrl);
    
        UIMap.Login();
        UIMap.ValidateLogonSuccess();
        // ...
        UIMap.LogOff();
        UIMap.ValidateLogoffSucceeded();
    
        window.Close();
    }

Pretty straight forward. Except that the call to “window.Close();” closed the browser, but not the ChromeDriver command window – so the test never terminated.

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-vG1XGLrHts8/UuuSGdgHH7I/AAAAAAAABOQ/wTnFZ5Ewv1k/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-4hs5IJmraW8/UuuSFTZDWzI/AAAAAAAABOI/XdqQxWeJumg/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->

So even though the test passes, I have to manually close the command window before the test run itself terminates.

After playing around a bit, I came up with this code to kill the command window (this works for Chrome – haven’t tested it for Firefox). Just replace “window.Close()” with this code:

    // kill the process so we don't lock up - for some reason window.Close() locks
    window.Process.Kill();
    
    var procs = Process.GetProcesses().Where(p =&gt; p.ProcessName.ToLower().Contains("chromedriver"));
    foreach (var p in procs)
    {
        p.Kill();
    }

That did it nicely for me (and made my Build-Deploy-Test workflows in my lab terminate correctly).

Happy (cross-browser) testing!

