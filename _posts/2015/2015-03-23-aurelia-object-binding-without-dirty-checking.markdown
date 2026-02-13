---
layout: post
title: 'Aurelia: Object Binding Without Dirty Checking'
date: '2015-03-23 21:31:53'
tags:
- development
---

Over the past few weeks I have been developing a Web UI using [Aurelia](http://aurelia.io/) by [Rob Eisenberg](https://twitter.com/eisenbergeffect). It’s really well thought out – though it’s got a steep learning curve at the moment since the documentation is still very sparse. Of course it hasn’t officially released yet, so that’s understandable!

## TypeScript

I love [TypeScript](http://www.typescriptlang.org/) – if it wasn’t for TypeScript, I would really hate Javascript development! Aurelia is written in [ES6](https://github.com/lukehoban/es6features) and ES7 which is [transpiled to ES5](https://babeljs.io/). You can easily write Aurelia apps in TypeScript – you can [transpile in Gulp](https://www.npmjs.com/package/gulp-typescript) if you want to – otherwise Visual Studio will transpile to Javascript for you anyway. Since I use TypeScript, I also use [Mike Graham’s TypeScript Aurelia sample repos](https://github.com/cmichaelgraham/aurelia-typescript). He has some great samples there if you’re just getting started with Aurelia/TypeScript. Code for this post comes from the “aurelia-vs-ts” solution in that repo.

## Binding in Aurelia

Aurelia has many powerful features out the box – and most of its components are pluggable too – so you can switch out components as and when you need to. Aurelia allows you to separate the view (html) from the view-model (a Javascript class). When you load a view, Aurelia binds the properties of the view-model with the components in the view. This works beautifully for primitives – Aurelia knows how to create a binding between an HTML element (or property) and the object property. Let’s look at home.html and home.ts to see how this works:

    &lt;template&gt;
      &lt;section&gt;
        &lt;h2&gt;${heading}&lt;/h2&gt;
    
        &lt;form role="form" submit.delegate="welcome()"&gt;
          &lt;div class="form-group"&gt;
            &lt;label for="fn"&gt;First Name&lt;/label&gt;
            &lt;input type="text" value.bind="firstName" class="form-control" id="fn" placeholder="first name"&gt;
          &lt;/div&gt;
          &lt;div class="form-group"&gt;
            &lt;label for="ln"&gt;Password&lt;/label&gt;
            &lt;input type="text" value.bind="lastName" class="form-control" id="ln" placeholder="last name"&gt;
          &lt;/div&gt;
          &lt;div class="form-group"&gt;
            &lt;label&gt;Full Name&lt;/label&gt;
            &lt;p class="help-block"&gt;${fullName | upper}&lt;/p&gt;
          &lt;/div&gt;
          &lt;button type="submit" class="btn btn-default"&gt;Submit&lt;/button&gt;
        &lt;/form&gt;
      &lt;/section&gt;
    &lt;/template&gt;

This is the view (html) for the home page (views\home.html). You bind to variables in the view-model using the ${var} syntax (lines 3 and 16). You can also bind attributes directly – like value.bind=”firstName” in line 8 binds the value of the input box to the “firstName” property. Line 16 uses a value converter to convert the value of the bound parameter to uppercase. Line 5 binds a function to the submit action. I don’t want to get into all the Aurelia binding capabilities here – that’s for another discussion.

Here’s the view-model (views\home.ts):

    export class Home {
        public heading: string;
        public firstName: string;
        public lastName: string;
    
        constructor() {
            this.heading = "Welcome to Aurelia!";
            this.firstName = "John";
            this.lastName = "Doe";
        }
    
        get fullName() {
            return this.firstName + " " + this.lastName;
        }
    
        welcome() {
            alert("Welcome, " + this.fullName + "!");
        }
    }
    
    export class UpperValueConverter {
        toView(value) {
            return value &amp;&amp; value.toUpperCase();
        }
    }
    

The code is very succinct – and easy to test. Notice the absence of any “binding plumbing”. So how does the html know to update when values in the view-model change? (If you’ve ever used [Knockout](http://knockoutjs.com/) you’ll be wondering where the observables are!)

## Dirty Binding

The bindings for heading, firstName and lastName are primitive bindings – in other words, when Aurelia binds the html to the property, it creates an observer on the property so that when the property is changed, a notification of the change is triggered. It’s all done under the covers for you so you can just assume that any primitive on any model will trigger change notifications to anything bound to them.

However, if you’re not using a primitive, then Aurelia has to fall-back on “dirty binding”. Essentially it sets up a polling on the object ([every 120ms](https://github.com/aurelia/binding/issues/37)). You’ll see this if you put a console.debug into the getter method:

    get fullName() {
        console.debug("Getting fullName");
        return this.firstName + " " + this.lastName;
    }

Here’s what the console looks like when you browse (the console just keeps logging forever and ever):

<!--kg-card-begin: html-->[![image](/assets/images/files/ac40cfab-c6ba-443d-a71b-c3f352680c44.png "image")](/assets/images/files/8acee728-be52-4af3-b44d-1d1a26a9fdd3.png)<!--kg-card-end: html-->

Unfortunately there simply isn’t an easy way around this problem.

## Declaring Dependencies

[Jeremy Danyow](https://github.com/jdanyow) did however leverage the pluggability of Aurelia and wrote a plugin for observing computed properties without dirty checking called [aurelia-computed](https://github.com/jdanyow/aurelia-computed). This is now incorporated &nbsp;into Aurelia and is plugged in by default.

This plugin allows you to specify dependencies explicitly – thereby circumventing the need to dirty check. Here are the changes we need to make:

1. Add a definition for the declarePropertyDependencies() method in Aurelia.d.ts (only necessary for TypeScript)
2. Add an import to get the aurelia-binding libs
3. Register the dependency

Add these lines to the bottom of the aurelia.d.ts file (in the typings\aurelia folder):

    declare module "aurelia-binding" {
        function declarePropertyDependencies(moduleType: any, propName: string, deps: any[]): void;
    }

This just lets Visual Studio know about the function for compilation purposes.

Now change home.ts to look as follows:

    import aub = require("aurelia-binding");
    
    export class Home {
        public heading: string;
        public firstName: string;
        public lastName: string;
    
        constructor() {
            this.heading = "Welcome to Aurelia!";
            this.firstName = "John";
            this.lastName = "Doe";
        }
    
        get fullName() {
            console.debug("Getting fullName");
            return this.firstName + " " + this.lastName;
        }
    
        welcome() {
            alert("Welcome, " + this.fullName + "!");
        }
    }
    
    aub.declarePropertyDependencies(Home, "fullName", ["firstName", "lastName"]);
    
    export class UpperValueConverter {
        toView(value) {
            return value &amp;&amp; value.toUpperCase();
        }
    }
    

The highlighted lines are the lines I added in. Line 24 is the important line – this explicitly registers a dependency on the “fullName” property of the Home class – on “firstName” and “lastName”. Now any time either firstName or lastName changes, the value of “fullName” is recalculated. Bye-bye polling!

Here’s the console output now:

<!--kg-card-begin: html-->[![image](/assets/images/files/64c1c8eb-ed5d-42b2-b9ca-98da6ea7c8f2.png "image")](/assets/images/files/79850200-d5b2-41d3-961e-16eee681f751.png)<!--kg-card-end: html-->

We can see that the fullName getter is called 4 times. This is a lot better than polling the value every 120ms. (I’m not sure why it’s called 4 times – probably to do with how the binding is initially set up. Both firstName and lastName change when the page loads and they are instantiated to “John” and “Doe” so I would expect to see a couple firings of the getter function at least).

## Binding to an Object

So we’re ok to bind to primitives – but we get stuck again when we want to bind to objects. Let’s take a look at app-state.ts (in the scripts folder):

    import aur = require("aurelia-router");
    
    export class Redirect implements aur.INavigationCommand {
        public url: string;
        public shouldContinueProcessing: boolean;
    
        /**
          * Application redirect (works with approuter instead of current child router)
          *
          * @url the url to navigate to (ex: "#/home")
          */
        constructor(url) {
            this.url = url;
            this.shouldContinueProcessing = false;
        }
    
        navigate(appRouter) {
            appRouter.navigate(this.url, { trigger: true, replace: true });
        }
    }
    
    class AppState {
        public isAuthenticated: boolean;
        public userName: string;
    
        /**
          * Simple application state
          *
          */
        constructor() {
            this.isAuthenticated = false;
        }
    
        login(username: string, password: string): boolean {
            if (username == "Admin" &amp;&amp; password == "xxx") {
                this.isAuthenticated = true;
                this.userName = "Admin";
                return true;
            }
            this.logout();
            return false;
        }
    
        logout() {
            this.isAuthenticated = false;
            this.userName = "";
        }
    }
    
    var appState = new AppState();
    export var state = appState;
    

The AppState is a static global object that tracks the state of the application. This is a good place to track logged in user, for example. I’ve added in the highlighted lines so that we can expose AppState.userName. Let’s open nav-bar.ts (in views\controls) and add a getter so that the nav-bar can display the logged in user’s name:

    import auf = require("aurelia-framework");
    import aps = require("scripts/app-state");
    
    export class NavBar {
        static metadata = auf.Behavior.withProperty("router");
    
        get userName() {
            console.debug("Getting userName");
            return aps.state.userName;
        }
    }

We can now bind to userName in the nav-bar.html view:

    &lt;template&gt;
      &lt;nav class="navbar navbar-default navbar-fixed-top" role="navigation"&gt;
        &lt;div class="navbar-header"&gt;
          &lt;button type="button" class="navbar-toggle" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1"&gt;
            &lt;span class="sr-only"&gt;Toggle Navigation&lt;/span&gt;
            &lt;span class="icon-bar"&gt;&lt;/span&gt;
            &lt;span class="icon-bar"&gt;&lt;/span&gt;
            &lt;span class="icon-bar"&gt;&lt;/span&gt;
          &lt;/button&gt;
          &lt;a class="navbar-brand" href="#"&gt;
            &lt;i class="fa fa-home"&gt;&lt;/i&gt;
            &lt;span&gt;${router.title}&lt;/span&gt;
          &lt;/a&gt;
        &lt;/div&gt;
    
        &lt;div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1"&gt;
          &lt;ul class="nav navbar-nav"&gt;
            &lt;li repeat.for="row of router.navigation" class="${row.isActive ? 'active' : ''}"&gt;
              &lt;a href.bind="row.href"&gt;${row.title}&lt;/a&gt;
            &lt;/li&gt;
          &lt;/ul&gt;
    
          &lt;ul class="nav navbar-nav navbar-right"&gt;
            &lt;li&gt;&lt;a href="#"&gt;${userName}&lt;/a&gt;&lt;/li&gt;
            &lt;li class="loader" if.bind="router.isNavigating"&gt;
              &lt;i class="fa fa-spinner fa-spin fa-2x"&gt;&lt;/i&gt;
            &lt;/li&gt;
          &lt;/ul&gt;
        &lt;/div&gt;
      &lt;/nav&gt;
    &lt;/template&gt;

I’ve added line 24. Of course we’ll see polling if we run the solution as is. So we can just declare the dependency, right? Let’s try it:

    import auf = require("aurelia-framework");
    import aub = require("aurelia-binding");
    import aps = require("scripts/app-state");
    
    export class NavBar {
        static metadata = auf.Behavior.withProperty("router");
    
        get userName() {
            return aps.state.userName;
        }
    }
    
    aub.declarePropertyDependencies(NavBar, "userName", [aps.state.userName]);

Seems to compile and run – but the value of userName is never updated!

It turns out that we can only declare dependencies to the same object (and only to primitives) using declarePropertyDependencies. Seems like we’re stuck.

## The Multi-Observer

I posed this question on the [gitter discussion page for Aurelia](https://gitter.im/Aurelia/Discuss). The guys working on Aurelia (and the community) are very active there – I’ve been able to ask Rob Eisenberg himself questions! Jeremy Danyow is also active on there (as is Mike Graham) so getting help is usually quick. Jeremy quickly verified that declarePropertyDependencies cannot register dependencies on other objects. However, he promptly whacked out the “Multi-Observer”. Here’s the TypeScript for the class:

    import auf = require("aurelia-framework");
    
    export class MultiObserver {
        static inject = [auf.ObserverLocator];
    
        constructor(private observerLocator: auf.ObserverLocator) {
        }
    
        /**
         * Set up dependencies on an arbitrary object.
         * 
         * @param properties the properties to observe
         * @param callback the callback to fire when one of the properties changes
         * 
         * Example:
         * export class App {
         * static inject() { return [MultiObserver]; }
         * constructor(multiObserver) {
         * var session = {
         * fullName: 'John Doe',
         * User: {
         * firstName: 'John',
         * lastName: 'Doe'
         * }
         * };
         * this.session = session;
         *
         * var disposeFunction = multiObserver.observe(
         * [[session.User, 'firstName'], [session.User, 'lastName']],
         * () =&gt; session.fullName = session.User.firstName + ' ' + session.User.lastName);
         * }
         * }
         */
        observe(properties, callback) {
            var subscriptions = [], i = properties.length, object, propertyName;
            while (i--) {
                object = properties[i][0];
                propertyName = properties[i][1];
                subscriptions.push(this.observerLocator.getObserver(object, propertyName).subscribe(callback));
            }
    
            // return dispose function
            return () =&gt; {
                while (subscriptions.length) {
                    subscriptions.pop()();
                }
            }
        }
    }

Add this file to a new folder called “utils” under “views”. To get this to compile, you have to add this definition to the aurelia.d.ts file (inside the aurelia-framework module declaration):

    interface IObserver {
        subscribe(callback: Function): void;
    }
    
    class ObserverLocator {
        getObserver(object: any, propertyName: string): IObserver;
    }

Now we can use the multi-observer to register a callback when any property on any object changes. Let’s do this in the nav-bar.ts file:

    import auf = require("aurelia-framework");
    import aub = require("aurelia-binding");
    import aps = require("scripts/app-state");
    import muo = require("views/utils/multi-observer");
    
    export class NavBar {
        static metadata = auf.Behavior.withProperty("router");
        static inject = [muo.MultiObserver];
    
        dispose: () =&gt; void;
        userName: string;
    
        constructor(multiObserver: muo.MultiObserver) {
            // set up a dependency on the session router object
            this.dispose = multiObserver.observe([[aps.state, "userName"]],() =&gt; {
                console.debug("Setting new value for userName");
                this.userName = aps.state.userName;
            });
        }
    
        deactivate() {
            this.dispose();
        }
    }

We register the function to execute when the value of the property on the object changes – we can execute whatever code we want in this callback.

Here’s the console after logging in:

<!--kg-card-begin: html-->[![image](/assets/images/files/8e3cd17e-5009-4696-a620-22534f09ba62.png "image")](/assets/images/files/a26b14ee-c3c9-4c15-8178-b751902c6100.png)<!--kg-card-end: html-->

There’s no polling – the view-model is bound to the userName primitive on the view-model. But whenever the value of userName on the global state object changes, we get to update the value. We’ve successfully avoided the dirty checking!

One last note: we register the dependency callback into a function object called “dispose”. We can then simply call this function when we want to unregister the callback (to free up resources). I’ve put the call in the deactivate() method, which is the method Aurelia calls on the view-model when navigating away from it. In this case it’s not really necessary, since the nav-bar is “global” and we won’t navigate away from it. But if you use the multi-observer in a view-model that is going to be unloaded (or navigated away from), be sure to put the dispose function somewhere sensible.

A big thank you to Jeremy Danyow for his help!

Happy binding!

