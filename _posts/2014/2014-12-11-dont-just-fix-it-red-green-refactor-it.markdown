---
layout: post
title: Don’t Just Fix It, Red-Green Refactor It!
date: '2014-12-11 21:04:30'
tags:
- development
---

I’m back to doing some dev again – for a real-life, going-to-charge-for application! It’s great to be based from home again and to be on some very cutting edge dev.

I’m very comfortable with [ASP.NET MVC](http://www.asp.net/mvc), but this project is the first [Nancy](http://nancyfx.org/) project I’ve worked on. We’re also using [Git on VSO](http://www.visualstudio.com/en-us/get-started/share-your-code-in-git-vs.aspx) for source control and backlogs, [MyGet](http://myget.org) to host internal [NuGet](https://www.nuget.org/) packages, [Octopus deploy](https://octopusdeploy.com/) for deployment, [Python](http://www.hanselman.com/blog/OneOfMicrosoftsBestKeptSecretsPythonToolsForVisualStudioPTVS.aspx) (with various libs, of course!) for number crunching and [Azure](http://azure.microsoft.com) to host VMs and websites (which are monitored with [AppInsights](http://msdn.microsoft.com/en-us/library/dn481095.aspx)). All in all it’s starting to shape up to a very cool application – details to follow as we approach go-live (play mysterious music here)…

## Ho Hum Dev

Ever get into a groove that’s almost too automatic? Ever been driving home and you arrive and think, “Wait a minute – how did I get here?”. You were thinking so intently on something else that you just drove “on automatic” without really paying attention to what you were doing.

Dev can sometimes get into this kind of groove. I was doing some coding a few days ago and almost missed a good quality improvement opportunity – fortunately, I was able to look up long enough to see a better way to do things, and hopefully save myself some pain down the line.

I was debugging some code, and something wasn’t working the way I expected. Here’s a code snippet showing two properties I was working with:

    protected string _name;
    public string Name
    {
        get 
        {
            if (string.IsNullOrEmpty(_name))
            {
                SplitKey();
            }
            return _name;
        }
        set 
        {
            _name = value;
            CombineKey();
        }
    }
    
    protected ComponentType _type;
    public string Type
    {
        get
        {
            return _type.ToString();
        }
        set
        {
            _type = ParseTypeEnum(value);
            CombineKey();
        }
    }

See how the getter for the Type property doesn’t match the code for the getter for Name? Even though I have unit tests for this getter, the tests are all passing!

Now the simple thing to do would have been to simply add the missing call to SplitKey() and carry on – but I wanted to know why the tests weren’t failing. I knew there were issues with the code (I had hit them while debugging) so I decided to take a step back and try some good practices: namely red-green refactor.

## Working with Quality in Mind

When you’re coding you should be working with quality in mind – that’s why I love unit testing so much. If you’re doing dev without unit testing, you’re only setting yourself up for long hours of painful in-production debugging. Not fun. Build with quality up front – while it may _feel_ like it’s taking longer to deliver, [you’ll save time in the long run](/why-you-absolutely-need-to-unit-test) since you’ll be adding new features instead of debugging poor quality code.

Here’s what you \*should\* be doing when you come across “hanky” code:

1. Do some coding
2. While running / debugging, find some bug
3. BEFORE FIXING THE BUG, write a FAILING unit test that exposes the bug
4. Refactor/fix till the test passes

So I opened up the tests for this entity and found the issue: I was only testing one scenario. This highlights that while code coverage is important, it can give you a false sense of security!

Here’s the original test:

    [TestMethod]
    public void SetsPropertiesCorrectlyFromKeys()
    {
        var component = new Component()
        {
            Key = "Logger_Log1"
        };
    
        Assert.AreEqual("Logger", component.Type);
        Assert.AreEqual("Log1", component.Name);
    }

ComponentType comes from an enumeration – and since Logger is the 1st value in the enum, it defaults to Logger if you don’t explicitly set the value. So while I had a test that was covering the entire method, it wasn’t testing all the combinations!

So I added a new test:

    [TestMethod]
    public void SetsPropertiesCorrectlyFromKeys2()
    {
        var component = new Component()
        {
            Key = "Service_S0"
        };
    
        Assert.AreEqual("Service", component.Type);
        Assert.AreEqual("S0", component.Name);
    }

Now when I ran the tests, the 2nd test failed. Excellent! Now I’ve got a further test that will check for a bad piece of code.

To fix the bug, I had to add another enum value and of course, add in the missing SplitKey() call in the Type property getter:

    public enum ComponentType
    {
        Unknown,
        Logger,
        Service
    }
    
    ...
    
    protected ComponentType _type;
    public string Type
    {
        get
        {
            if (_type == ComponentType.Unknown)
            {
                SplitKey();
            }
            return _type.ToString();
        }
        set
        {
            _type = ParseTypeEnum(value);
            CombineKey();
        }
    }

Now both tests are passing. Hooray!

## Conclusion

I realize the red-green refactoring isn’t a new concept – but I wanted to show a real-life example of how you should be thinking about your dev and debugging. Even though the code itself had 100% code coverage, there were still bugs. But debugging with quality in mind means you can add tests that cover specific scenarios – and which will reduce the amount of buggy code going into production.

Happy dev’ing!

