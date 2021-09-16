---
layout: post
title: Failing Builds based on Code Coverage
date: '2013-02-09 01:01:00'
tags:
- build
---

The logical next step after you start unit testing your code is to analyse code coverage. You can do this easily in TFS by enabling Code Coverage in the test settings of the default build template. But what about failing builds (or checkins) based on low code coverage?

Out of the box, there’s nothing that can do that. You could write a checkin policy that inspects code coverage, but you’d have to make sure it’s deployed to all VS instances. Or, you could implement the logic in a build, and then make the build a rolling gated checkin build. That way if the build fails for low coverage, the checkin is discarded.

## Modifying the Default Template

I’ve created a CodeActivity for your toolbox that will inspect the test coverage and return the coverage percentage. You can then easily implement some logic from there.

To get started, you’ll have to import the Custom assembly (link at the bottom of this post). Follow steps 1 to 3 of my [previous custom build post](http://colinsalmcorner.blogspot.com/2013/02/custom-build-task-include-merged.html) to get going.

Now you can add the ColinsALMCorner.CustomBuildTasks in to the imports of your workflow:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-nSLjQI64yqU/URUTAn7hq-I/AAAAAAAAAnc/ApNLrZoXdlo/image_thumb.png?imgmax=800 "image")](http://lh4.ggpht.com/-uFYwr-eopiI/URUS_LxqDYI/AAAAAAAAAnU/zxwdE1TfSqA/s1600-h/image%25255B2%25255D.png)<!--kg-card-end: html-->

Click on the Arguments tab and add an Int32 in argument called “CoverageFailureIfBelow”. Set the metadata as follows:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-pPjB5feSLkA/URUTEF5clcI/AAAAAAAAAns/D71dF3t2yok/image_thumb%25255B2%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-ydfIeHUXePk/URUTBhLTkGI/AAAAAAAAAnk/eIEBsALcjiY/s1600-h/image%25255B6%25255D.png)<!--kg-card-end: html-->

Now scroll down in the workflow until **after** the testing section, between “If CompilationStatus = Uknown” and “If TestingStatus = Uknown”. Add a Sequence between the two If activities called “Check Coverage”.

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-jfWeLc0BmvA/URUTIP_I1QI/AAAAAAAAAoA/CQQpNfXpOB4/image_thumb%25255B4%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-ajVXY2oML4U/URUTFWhpdlI/AAAAAAAAAn0/14bzM6-jTh0/s1600-h/image%25255B10%25255D.png)<!--kg-card-end: html-->

Click on Variables at the bottom of the workflow window and create an Int32 variable (scoped to Check Coverage, called coverageTotal). You can double-click the sequence to “zoom in”.

The “GetCoverageTotal” activity relies on the test attachments that the test engine uploads to TFS when the tests are completed (most notably the Code Coverage attachment). Since this is done asynchronously, I introduced a Delay activity. Add a new Argument to the workflow (if you don’t want to hard-code the delay) and set its default to 10 seconds.

Set the Timeout of the Delay activity to

<!--kg-card-begin: html--><font face="Courier New">Timespan.FromSeconds(arg)</font><!--kg-card-end: html-->

where _arg_ is the name of your timeout argument.

Drop a “GetCoverageTotal” activity (from the custom assembly) into the sequence (NOTE: You may have to import the assembly into the Toolbox if you’ve never done that before). Set the properties as follows:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-YeCIihP6MAM/URUTKlUyRbI/AAAAAAAAAoQ/2UuYhVr6ek0/image_thumb%25255B14%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-xtYLqShGV-A/URUTJNV5wuI/AAAAAAAAAoI/7f8eS6KBBU8/s1600-h/image%25255B30%25255D.png)<!--kg-card-end: html-->

Once that’s done, you now have the coverage total, so you can do whatever you need to. Here’s what I did:

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-TZ6t_0yDqh0/URUTNQab3lI/AAAAAAAAAog/ReNMBCE2zPw/image_thumb%25255B16%25255D.png?imgmax=800 "image")](http://lh5.ggpht.com/-34TlW0CDXYw/URUTL_MsdzI/AAAAAAAAAoY/_00-xufRTq8/s1600-h/image%25255B34%25255D.png)<!--kg-card-end: html-->

I put an If that evaluates if the total coverage is too low or not. If the coverage is too low, I set the BuildDetail.TestStatus to TestStatus.Failed. I also set the overall build status to PartiallySucceeded. I then have a WriteBuildWarning (“Failing build because coverage is too low”) or WriteBuildMessage (“Coverage is acceptable”) depending on the result.

Now I just queue up a build, and voila – I can now fail it for low coverage (or in this case, set the result to Partially Successful).

<!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-Xs3LDlV2Bsk/URUTTg3K0rI/AAAAAAAAAow/ba5PAMSuyy0/image_thumb%25255B12%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-o2UxodX4KM4/URUTOtdjo0I/AAAAAAAAAoo/ybzXtjAjo-o/s1600-h/image%25255B26%25255D.png)<!--kg-card-end: html-->

You can download the dll and the DefaultCoverageTemplate.11.1.xaml from my [skydrive](http://sdrv.ms/XsMf8h).

Happy coverage!

