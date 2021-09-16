---
layout: post
title: GenericAutomationPeer – Helping the Coded UI Framework Find Your Custom Controls
date: '2011-11-03 22:21:00'
tags:
- testing
---

Sometimes you’ll write an WPF application that has some sort of “dynamic” way of loading portions of the UI (think: [Prism](http://compositewpf.codeplex.com/)). Sometimes entire frameworks are too much, so you’d prefer something a bit simpler – like, say, a TabControl with a data template. Bind the ItemsSource of your TabControl to an ObservableCollection (where T is some model) and you’ve got a “dynamic” interface.

So you’ve written your killer dynamic interface app – and, since you’re a good developer, you try to add some coded UI tests.

**<u>Problem:</u>** The UI Test framework doesn’t “see” any of the controls that were loaded dynamically! What gives?

<!--kg-card-begin: html-->[![clip_image001](http://lh6.ggpht.com/-o4LLkVVhXOo/TrKVXdg2dTI/AAAAAAAAAUg/5RPUOBZq-p4/clip_image001_thumb.jpg?imgmax=800 "clip\_image001")](http://lh6.ggpht.com/-DcFLf0Uxges/TrKVV0HYxyI/AAAAAAAAAUY/aXxzDeswSXY/s1600-h/clip_image0013.jpg)<!--kg-card-end: html-->

_The figure above shows the “deepest” level that the UI Framework can see – none of the child controls (the comboboxes or table) exist as far as the coded UI Test framework is concerned._

What’s happening here is that the [AutomationPeer](http://msdn.microsoft.com/en-us/library/cc165614.aspx) of the TabControl doesn’t know anything about the controls within it, since they’re loaded dynamically. You have to help your little TabControl a bit. You have to let the Automation framework know about all the little controls that you load. But what if each tab loads a completely different UserControl? This sounds like a lot of work…

## The Solution: GenericAutomationPeer

Fortunately, you can walk the child controls and just get them to give their “default” AutomationPeers to you (most primitive WPF controls – like TextBoxes, ComboBoxes, Buttons and so on – have built in AutomationPeers). So we need 2 things: a way to walk the child controls and a hook into the Automation Framework.

      public class GenericAutomationPeer : UIElementAutomationPeer {
        public GenericAutomationPeer(UIElement owner) : base(owner) { }
        
        protected override List<automationpeer> GetChildrenCore() {
          var list = base.GetChildrenCore();
          list.AddRange(GetChildPeers(Owner));
          return list;
        }
        
        private List<automationpeer> GetChildPeers(UIElement element) {
          var list = new List<automationpeer>();
          for (int i = 0; i &lt; VisualTreeHelper.GetChildrenCount(element); i++) {
            var child = VisualTreeHelper.GetChild(element, i) as UIElement;
            if (child != null) {
              var childPeer = UIElementAutomationPeer.CreatePeerForElement(child);
              if (childPeer != null) {
                list.Add(childPeer);
              } else { 
                list.AddRange(GetChildPeers(child));
              }
            }
          }
          return list;
          }
        }
      </automationpeer></automationpeer></automationpeer>

\>p\>This class inherits from UIElementAutomationPeer, so you just need to override the GetChildrenCore() method. Inside that, just use the VisualTreeHelper to walk the child controls. For each child control, see if it has an AutomationPeer by calling the static method UIElementAutomationPeer.CreatePeerForElement(). If it has an element, add it to the list of AutomationPeers. If it doesn’t, then recursively call to see if it’s children have AutomationPeers.

So we’ve got our GenericAutomationPeer: now we just need a hook in to use it. In this example, the “lowest” control visible to the Automation Framework was the TabControl – so that’s where we’ll get our hook in.

    public class CustomTabControl : TabControl{ protected override AutomationPeer OnCreateAutomationPeer() { return new GenericAutomationPeer(this); }}

We create a CustomTabControl that inherits from TabControl and simply overrides its OnCreateAutomationPeer. Inside the override, simple new up a GenericAutomationPeer, and you're done. Don’t forget to change the XAML from TabControl to local:CustomTableControl (where local is your imported namespace).

## Other Tools for your Toolbox

- [UISpy](http://msdn.microsoft.com/en-us/library/ms727247.aspx) – this lets you inspect exactly what the Automation Framework can “see”
- [CUITE](http://cuite.codeplex.com/) – Coded UI Test Enhancements on Codeplex – useful coded UI Extension library

Happy (coded UI) testing!

**Update:** This [post](http://social.msdn.microsoft.com/forums/en-US/wpf/thread/fa8eb86f-5001-4af6-adb3-ceb0799a0cf3) also had a solution specifically for TabControls that avoids having to implement AutomationPeers entirely.

