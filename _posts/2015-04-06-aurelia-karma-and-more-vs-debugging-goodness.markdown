---
layout: post
title: Aurelia, Karma and More VS Debugging Goodness
date: '2015-04-06 23:56:47'
tags:
- development
- testing
---

In my [previous post](/aurelia--debugging-from-within-visual-studio) I walked through how to change [Aurelia](http://aurelia.io/) to load modules via [Require.js](http://requirejs.org/) so that you can set breakpoints and debug from VS when you run your Aurelia project. In this post I want to share some tips about unit testing your Aurelia view-models.

## Unit Testing Javascript

If you aren’t yet convinced of the value of unit testing, please read my [post about why you absolutely should be](/why-you-absolutely-need-to-unit-test). Unfortunately, unit testing Javascript in Visual Studio (and during automated builds) is a little harder to do than running unit tests on managed code. This post will show you some of the techniques I use to unit test Javascript in my Aurelia project – though of course you don’t need to be using Aurelia to make use of these techniques. If you want to see the code I’m using for this post, check out [this repo](https://github.com/colindembovsky/aurelia-appInsights).

### But I’ve already got tests!

This post isn’t going to go too much into _how to_ unit test – there are hundreds of posts about how to test. I’m going to assume that you already have some unit tests. I’ll discuss the following topics in this post:

- Basic Karma/Jasmine overview
- Configuring Karma and RequireJS
- Running Karma from Gulp
- Using a SpecRunner.html page to enable debugging unit tests
- Fudges to enable PhantomJS
- Code Coverage
- Running tests in your builds (using TeamBuild)
- Karma VS Test adapter

### Karma and Jasmine

There are many JavaScript testing frameworks out there. I like [Jasmine](http://jasmine.github.io/) as a ([BDD](http://en.wikipedia.org/wiki/Behavior-driven_development)) testing framework, and I like [Karma](http://karma-runner.github.io/0.12/index.html) (which used to be called Testacular) as a test runner. One of the things I like about Karma is that you can run your tests in several browsers – it also has numerous “reporters” that let you track the tests, and even add code coverage. Aurelia itself uses Karma for its testing.

### Configuring Karma and RequireJS

To configure karma, you have to set up a karma config file – by convention it’s usually called karma.conf.js. If you use [karma-cli](https://www.npmjs.com/package/karma-cli), you can run “[karma init](http://karma-runner.github.io/0.12/intro/configuration.html)” to get karma to lead you through a series of questions to help you set up a karma config file for the first time. I wanted to use requirejs, mostly because using requirejs means I can set breakpoints in Visual Studio and debug. So I made sure to answer “yes” for that question. Unfortunately, that opens a can of worms!

The reason for the “can of worms” is that karma tries to serve all the files necessary for the test run – but if they are AMD modules, then you can’t “serve” them – they need to be loaded by requirejs. In order to do that, we have to fudge the karma startup a little. We specify the files that should be served in the karma.conf.js file, being careful to “exclude” the files. This flag tells karma to serve the file when it is requested, but not to execute it (think of it as treating the file as static text rather than a JavaScript file to execute). Then, we create a “test-main.js” file to configure requirejs, load the modules and then launch karma.

Here’s the karma.conf.js file:

    // Karma configuration
    module.exports = function (config) {
        config.set({
            basePath: "",
    
            frameworks: ["jasmine", "requirejs", "sinon"],
    
            // list of files / patterns to load in the browser
            files: [
                // test specific files
                "test-main.js",
                "node_modules/jasmine-sinon/lib/jasmine-sinon.js",
    
                // source files
                { pattern: "dist/**/*.js", included: false },
    
                // test files
                { pattern: 'test/unit/**/*.js', included: false },
    
                // framework and lib files
                { pattern: "Content/scripts/**/*.js", included: false },
            ],
    
            // list of files to exclude
            exclude: [
            ],
    
            // available reporters: https://npmjs.org/browse/keyword/karma-reporter
            reporters: ["progress"],
    
            // web server port
            port: 9876,
    
            // enable / disable colors in the output (reporters and logs)
            colors: true,
    
            // possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
            logLevel: config.LOG_DEBUG,
    
            // enable / disable watching file and executing tests whenever any file changes
            autoWatch: true,
    
            // available browser launchers: https://npmjs.org/browse/keyword/karma-launcher
            browsers: ["Chrome"],
    
            // Continuous Integration mode
            // if true, Karma captures browsers, runs the tests and exits
            singleRun: true
        });
    };

Notes:

- Line 6: We tell karma what frameworks to use when running the tests – jasmine (the test framework), requirejs (for loading modules) and sinon (for mocking). These are installed using “npm install [karma-jasmine|karma-requirejs|karma-sinon] respectively
- Lines 11/12: We load files that the tests will need – test-main to configure the modules for the test and the sinon file to load the sinon libs. Since these files are not “excluded”, karma executes them on load.
- Line 15: We serve all the source files we are testing, using the “exclude” to tell karma to serve them but not execute them (so it only serves them when requested – requirejs will load them)
- Line 18: We serve all the test (spec) files to run (again, not executing them)
- Line 21: We serve libraries (including the Aurelia framework)

Here’s the test-main.js file:

    var allTestFiles = [];
    var allSourceFiles = [];
    
    var TEST_REGEXP = /(spec|test)\.js$/i;
    var SRC_REGEXP = /dist\/[a-zA-Z]+\/[a-zA-Z]+.js$/im;
    
    var normalizePathToSpecFiles = function (path) {
        return path.replace(/^\/base\//, '').replace(/\.js$/, '');
    };
    
    var normalizePathToSourceFiles = function (path) {
        return path.replace(/^\/base\/dist\//, '').replace(/\.js$/, '');
    };
    
    var loadSourceModulesAndStartTest = function () {
        require(["aurelia/aurelia-bundle"], function () {
            require(allSourceFiles, function () {
                require(allTestFiles, function () {
                    window. __karma__.start();
                });
            });
        });
    };
    
    Object.keys(window. __karma__.files).forEach(function (file) {
        if (TEST_REGEXP.test(file)) {
            allTestFiles.push(normalizePathToSpecFiles(file));
        } else if (SRC_REGEXP.test(file)) {
            allSourceFiles.push(normalizePathToSourceFiles(file));
        }
    });
    
    require.config({
        // Karma serves files under /base, which is the basePath from your config file
        baseUrl: "/",
    
        paths: {
            test: "/base/test",
            dist: "/base/dist",
            views: "/base/dist/views",
            resources: "/base/dist/resources",
            aurelia: "/base/Content/scripts/aurelia",
        },
    
        // dynamically load all test files
        deps: ["aurelia/aurelia-bundle"],
    
        // we have to kickoff jasmine, as it is asynchronous
        callback: loadSourceModulesAndStartTest
    });

Notes:

- Line 4, 5: We set up regex patterns to match test (spec) files as well as source files
- Line 8, 12: We normalize the path to test or source files. This is necessary since the paths that requirejs use are a different to the base path that karma sets up.
- Lines 16-18: We load the modules we need in order of dependency – starting with Aurelia (frameworks), then the sources, and then the test files
- Line 19: We need to start the karma engine ourselves, since we’re hijacking the default start to load everything via requirejs
- Line 25: We hook into the karma function that loads files to normalize the file paths
- Line 37: We set up paths for requirejs
- Line 46: We tell requirejs that the most “basic” dependency is the Aurelia framework
- Line 49: We tell karma to execute our custom launch function once the “base” dependency is loaded

To be honest, figuring out the final path and normalize settings was a lot of trial and error. I turned karma logging onto debug, and then just played around until karma was serving all the files and requirejs was happy with path resolution. You’ll have to play around with these paths yourself for your project structure.

#### Running Karma Test from the CLI

Now we can run the karma tests: simply type “karma start” and karma will fire up and run the tests: you should see the Chrome window popping up (assuming you’re using the Chrome karma launcher) and a message telling you that the tests were run successfully.

### Running Karma from Gulp

Now that we have the tests running from the karma CLI, we can easily run them from within Gulp. We are using Gulp to transpile TypeScript to Javascript, compile LESS files to CSS and do minification and any other “production-izing” we need – so running tests in Gulp makes sense. Also, this way we make sure we’re using the latest sources we have instead of old stale code that’s been lying around (especially if you forget to run the gulp build tasks!). Here are the essential bits of the “unit-test” target in Gulp:

    var gulp = require('gulp');
    var karma = require("karma").server;
    
    gulp.task("unit-test", ["build-system"], function () {
        return karma.start({
            configFile: __dirname + "/../../karma.conf.js",
            singleRun: true
        });
    });

Notes:

- We import Gulp and Karma server. I didn’t install gulp-karma – rather, I just rely on “pure” karma.
- We create a task called “unit-test” that fist calls “build-system” before invoking karma
- The build-system task transpiles TypeScript to JavaScript – we make sure that we generate un-minified files and source maps in this task (so that later on we can set breakpoints and debug)
- We tell karma where to find the karma config file (so you need to specify the path relative to \_\_dirName, which is the current directory where the Gulp script is
- We tell karma to perform a single run, rather then keeping the browsers open and running the tests every time we change a file

We can now run “gulp unit-test” from the command line, or we can execute the gulp “unit-test” task from the Visual Studio Task Runner Explorer (which is native to VS 2015 and can be installed into VS 2013 [via an extension](https://visualstudiogallery.msdn.microsoft.com/8e1b4368-4afb-467a-bc13-9650572db708)):

<!--kg-card-begin: html-->[![image](/assets/images/files/c1a1b3d7-1335-4a1a-839f-d95243937e2c.png "image")](/assets/images/files/989fe25d-d1fb-4e2c-836e-b09eeb23b607.png)<!--kg-card-end: html-->
### Debugging Tests

Now that we can run the tests from Gulp, we may want to debug while testing. In order to do that, we’ll need to make sure the tests can run in IE (since VS will break on code running in IE). The karma launcher creates its own dynamic page to launch the tests, so we’re going to need to code an html page ourselves if we want to be able to debug tests. I create a “SpecRunner.html” page in my unit-test folder:

    &lt;!DOCTYPE html&gt;
    &lt;html&gt;
    &lt;head&gt;
        &lt;meta charset="utf-8"&gt;
        &lt;title&gt;Jasmine Spec Runner v2.2.0&lt;/title&gt;
    
        &lt;link rel="stylesheet" href="../../node_modules/jasmine-core/lib/jasmine-core/jasmine.css"&gt;
        &lt;script src="/Content/scripts/jquery-2.1.3.js"&gt;&lt;/script&gt;
        
        &lt;!-- source files here... --&gt;
        &lt;script src="/Content/scripts/core-js/client/core.js"&gt;&lt;/script&gt;
        &lt;script src="/Content/scripts/requirejs/require.js"&gt;&lt;/script&gt;
        
        &lt;script&gt;
            var baseUrl = window.location.origin;
            require.config({
                baseUrl: baseUrl + "/src",
                paths: {
                    jasmine: baseUrl + "/node_modules/jasmine-core/lib/jasmine-core/jasmine",
                    "jasmine-html": baseUrl + "/node_modules/jasmine-core/lib/jasmine-core/jasmine-html",
                    "jasmine-boot": baseUrl + "/node_modules/jasmine-core/lib/jasmine-core/boot",
                    "sinon": baseUrl + "/node_modules/sinon/pkg/sinon",
                    "jasmine-sinon": baseUrl + "/node_modules/jasmine-sinon/lib/jasmine-sinon",
                    aurelia: baseUrl + "/Content/scripts/aurelia",
                    webcomponentsjs: baseUrl + "/Content/scripts/webcomponentsjs",
                    dist: baseUrl + "/dist",
                    views: baseUrl + "/dist/views",
                    resources: baseUrl + "/dist/resources",
                    test: "/test"
                },
                shim: {
                    "jasmine-html": {
                        deps: ["jasmine"],
                    },
                    "jasmine-boot": {
                        deps: ["jasmine", "jasmine-html"]
                    }
                }
            });
    
            // load Aurelia and jasmine...
            require(["aurelia/aurelia-bundle"], function() {
                // ... then jasmine...
                require(["jasmine-boot"], function () {
                    // .. then jasmine plugins...
                    require(["sinon", "jasmine-sinon"], function () {
                        // build a list of specs
                        var specs = [];
                        specs.push("test/unit/aurelia-appInsights.spec");
    
                        // ... then load the specs
                        require(specs, function () {
                            // finally we can run jasmine
                            window.onload();
                        });
                    });
                });
            });
        &lt;/script&gt;
    &lt;/head&gt;
    
    &lt;body&gt;
    &lt;/body&gt;
    &lt;/html&gt;

Notes:

- Lines 15-39: We configure requirejs for the tests
- Lines 15: the base url is the window location – when debugging from VS this is usually [http://localhost](http://localhost) followed by some port
- Lines 19-29: the paths requirejs needs to resolve all the modules we want to load, as well as some jasmine-specific libs
- Lines 31-38: we need to shim a couple of jasmine libs to let requirejs know about their dependencies
- Lines 42, 44, 46: We load the dependencies in order so that requirejs loads them in the correct order
- Line 49: We create an array of all our test files
- Lines 52, 54: After loading the test specs, we trigger the onload() method which start the karma tests

Again you see that we hijack the usual Jasmine startup so that we can get requirejs to load all the sources, libs and tests before launching the test runner. Now we set the SpecRunner.html page to be the startup page for the project, and hit F5:

<!--kg-card-begin: html-->[![image](/assets/images/files/3c97596e-1d60-4f91-b7e0-ff1df45debda.png "image")](/assets/images/files/51bf3f4c-64f2-4f74-9dd3-7bf7c0825afa.png)<!--kg-card-end: html-->

Now that we can finally run the tests from VS in IE, we can set a breakpoint, hit F5 and we can debug!

<!--kg-card-begin: html--> [![image](/assets/images/files/6f30f6e2-d174-4283-9029-1b8945ebfd43.png "image")](/assets/images/files/49f3907d-044c-4be6-9b49-8260465651d8.png)<!--kg-card-end: html-->
### PhantomJS – mostly harmless\*, er, headless

While debugging in IE or launching Chrome from karma is great, there are situations where we may want to run our tests without the need for an actual browser (like on the build server). Fortunately there is a tool that allows you to run “headless” tests – PhantomJS. And even better – there’s a PhantomJS launcher for karma! Let’s add it in:

Run “npm install karma-phantomjs-launcher --save-dev” to install the PhantomJS launcher for karma. Then change the launcher config in the karma.conf.js file from [“Chrome”] to [“PhantomJS2”] and run karma. Unfortunately, this won’t work: you’ll likely see an error like this:

<!--kg-card-begin: html--><font size="2" face="Courier New">TypeError: 'undefined' is not a function (evaluating 'Array.prototype.forEach.call.bind(Array.prototype.forEach)')</font><!--kg-card-end: html-->

This sounds like a native JavaScript problem – perhaps since Aurelia uses ES6 (and even ES7) we need a more modern launcher. Let’s try install PhantomJS2 (the PhantomJS launcher that uses an experimental Phantom 2, a more modern version of PhantomJS). That seems to get us a little further:

<!--kg-card-begin: html--><font size="2" face="Courier New">ReferenceError: Can't find variable: Map</font><!--kg-card-end: html-->

Hmm. Map is again, an [ES6 structure](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map). Fortunately there is a library with the ES5 polyfills for some newer ES6 structures like Map: harmony-collections. We run “npm install harmony-collections --save-dev” to install the harmony-collections package, and then reference it in the karma.conf.js file (on line 13):

<!--kg-card-begin: html--><font size="2" face="Courier New">"node_modules/harmony-collections/harmony-collections.min.js",</font><!--kg-card-end: html-->

We get a bit further, but there is still something missing:

<!--kg-card-begin: html--><font size="2" face="Courier New">ReferenceError: Can't find variable: Promise</font><!--kg-card-end: html-->

Again a little bit of searching leads to another node package: so we run “npm install promise-polyfill --save-dev” and again reference the file (just after the harmony-collections reference):

<!--kg-card-begin: html--><font size="2" face="Courier New">"node_modules/promise-polyfill/Promise.min.js",</font><!--kg-card-end: html-->

Success! We can now run our tests headless.

In another system I was coding, I ran into a further problem with the “find” method on arrays. Fortunately, we can polyfill the find method too! I didn’t find a package for that – I simply added the polyfill from [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find) into one of my spec files.

### Code Coverage

So we can now run test from karma, from Gulp, from Task explorer, from VS using F5 and we can run them headless using PhantomJS2. If we add in a coverage reporter, we can even get some code coverage analysis: run “npm install karma-coverage --save-dev”. That will install a new reporter, so we need to add it in to the reporters section of karma.conf.js:

    reporters: ["progress", "coverage"],
    
    coverageReporter: {
        dir: "test/coverage/",
        reporters: [
            { type: 'lcov', subdir: 'report-lcov' },
            { type: 'text-summary', subdir: '.', file: 'coverage-summary.txt' },
            { type: 'text' },
        ]
    },

We add the reporter in (just after “progress”). We also configure what sort of coverage information formats we want and which directory the output should go to. Since the coverage requires our code to be instrumented, we need to add in a preprocessor (just above reporters):

    preprocessors: {
        "dist/**/*.js": ["coverage"]
    },

This tells the coverage engine to instrument all the js files in the dist folder. Any other files we want to calculate coverage from, we’ll need to add in the glob pattern.

<!--kg-card-begin: html-->[![image](/assets/images/files/4975c7b8-4961-4d9d-997f-68f2bcc6e6e6.png "image")](/assets/images/files/3ae2cabb-522c-4223-99e7-49c7ebe4852d.png)<!--kg-card-end: html-->

The output in the image above is from the “text” output. For more detailed coverage reports, we browse to test/coverage/report-lcov/lcov-report/index. We can then click through the folders and then down to the files, where we’ll be able to see exactly which lines our test covered (or missed):

<!--kg-card-begin: html-->[![image](/assets/images/files/a99f2622-c202-4a67-aac3-559fac6b0d1a.png "image")](/assets/images/files/c6445ebf-6d48-4466-9226-a930de914efd.png)<!--kg-card-end: html-->

This will help us discover more test opportunities.

### Running Test in TeamBuilds

With all the basics in place, we can easily include the unit tests into our builds. If you’re using TFS 2013 builds, you can just add a PowerShell script into your repo and then add that script as a pre- or post-test script. Inside the PowerShell you simply invoke “gulp unit-test” to run the unit tests via Gulp. I wanted to be a bit fancier, so I also added code to inspect the output from the test run and the coverage to add them into the build summary:

<!--kg-card-begin: html-->[![image](/assets/images/files/05970f61-df8b-4506-8973-cdc631cdc600.png "image")](/assets/images/files/a37d8fdf-bdfe-4899-8f4b-d7374328e5bb.png)<!--kg-card-end: html-->

The full PowerShell script is [here](https://github.com/colindembovsky/aurelia-appInsights/blob/master/RunGulpTests.ps1).

### Seeing Tests in Visual Studio

Finally, just in case we don’t have enough ways of running the tests, we can install the [Visual Studio Karma Test Adapter](https://visualstudiogallery.msdn.microsoft.com/4cd59e4a-82e8-4b4e-8302-d102fc81b090). This great adapter picks up the tests we’ve configured in karma and displays them in the test explorer window, where we can run them:

<!--kg-card-begin: html-->[![image](/assets/images/files/5e16a06b-31b9-499c-9c76-be792410de29.png "image")](/assets/images/files/db9e4f06-1ac0-40b8-bff4-15ab3edd2d34.png)<!--kg-card-end: html-->
## Conclusion

Unit testing your front-end view-model logic is essential if you’re going to deploy quality code. Enabling a good experience for unit testing requires a little bit of thought and some work – but once you’ve got the basics in place, you’ll be good to go. Ensuring quality as you code means you’ll have better quality down the road – and that means more time for new features and less time fixing bugs. Using Gulp and Karma enables continuous testing, and augmenting these with the techniques I’ve outlines you can also debug tests, run tests several ways and even integrate the tests (and coverage) into your builds.

Happy testing!

\* [Mostly Harmless](http://en.wikipedia.org/wiki/Mostly_Harmless) – from the Hitchhikers Guide to the Galaxy by Douglas Adams

