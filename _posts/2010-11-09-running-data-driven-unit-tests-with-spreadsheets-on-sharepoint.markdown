---
layout: post
title: Running Data-Driven Unit Tests with SpreadSheets on Sharepoint
date: '2010-11-09 22:38:00'
tags:
- build
---

Data-driven unit tests are a great way to run lots of tests. You define a test harness (or method) and tie it to a spreadsheet in an Excel file to “drive” the test – each row in the spreadsheet is one iteration of the test case. I am not going to go into any more detail here – there are plenty of examples about how to set these up. Another advantage of using Excel for data-driven tests is that testers love Excel. They can open and edit the spreadsheet and add / edit / remove test cases at will.

When a developer creates a test method, they can specify the spreadsheet to connect to using the [DataSource] attribute. Usually, you’d have a spreadsheet in your solution and set it to deploy in the test settings file (or by marking its properties to copy to output on a build) and then use the |DataDirectory| placeholder in the [DataSource] attribute to tell the test to look in the build output folder for the Excel file to open.

There are some drawbacks to doing this if your test team is maintaining the spreadsheets. They’ll need to have access to the source folder (out of Visual Studio) in order to change the spreadsheet. This is hardly ideal. A much better approach would be to set up a document library on a fileshare. Going one step better, set up a document library on Sharepoint – that way you get version history on the test data spreadsheets.

So if you decide to put the spreadsheets in a Sharepoint document library, you’ll have to decide on which site you want to store your docs on and then jump through a few hoops to hook up the tests to the Sharepoint documents. It makes sense to use the Team Project Portal for these sorts of documents, but any Sharepoint site will do.

Once you’ve uploaded your Excel files to a document library (don’t forget to enable Versioning if you want to track changes to these files), you have to do 2 things to get the test methods to work.

Firstly, you’ll have to map a network drive to the document library in the [ClassInitialize] method of your test class. This maps the document library of sharepoint to a local drive letter. You need to do this programmatically since the mapping won’t persist after you log out. Make sure that the account that is running the build controller has at least read access to the document library! I downloaded (and tweaked) [this project from CodeProject](http://www.codeproject.com/KB/system/mapnetdrive.aspx) to get some code for mapping network drives. I then created a method that map the drive:

<!--kg-card-begin: html--><font face="Consolas"><font size="2"><span><font color="#0000ff">public</font></span><font color="#000000"> </font><span><font color="#0000ff">static</font></span><font color="#000000"> </font><span><font color="#0000ff">void</font></span><font color="#000000"> MapNetworkDrive(</font><span><font color="#0000ff">string</font></span><font color="#000000"> localDrive, </font><span><font color="#0000ff">string</font></span></font></font><!--kg-card-end: html--><!--kg-card-begin: html--><font face="Consolas"><font size="2"><font color="#000000"> serverDrive)<br>{<br>    </font><span><font color="#0000ff">var</font></span><font color="#000000"> drive = </font><span><font color="#0000ff">new</font></span><font color="#000000"> </font><span><font color="#2b91af">NetworkDrive</font></span></font></font><!--kg-card-end: html--><!--kg-card-begin: html--><font face="Consolas"><font size="2"><font color="#000000">();<br>    drive.PromptForCredentials = </font><span><font color="#0000ff">false</font></span></font></font><!--kg-card-end: html--><!--kg-card-begin: html--><font face="Consolas"><font size="2"><font color="#000000">;<br>    drive.ShareName = serverDrive;<br>    drive.LocalDrive = localDrive;<br>    drive.Force = </font><span><font color="#0000ff">true</font></span></font><font color="#000000" size="2">;<br>    drive.MapDrive();<br>}</font></font><!--kg-card-end: html-->

I’m telling the library no to prompt for credentials and setting Force to true will unmap if the mapping exists already. You can then call this method from the [ClassInitialize] method:

    <font face="Consolas"><font size="2"><font color="#000000">[</font><span><font color="#2b91af">ClassInitialize</font></span></font></font><font face="Consolas"><font size="2"><font color="#000000">]<br></font><span><font color="#0000ff">public</font></span><font color="#000000">&nbsp;</font><span><font color="#0000ff">static</font></span><font color="#000000">&nbsp;</font><span><font color="#0000ff">void</font></span><font color="#000000"> Setup(</font><span><font color="#2b91af">TestContext</font></span></font></font><font face="Consolas"><font size="2"><font color="#000000"> context)<br>{<br></font><span><font color="#2b91af"> NetworkDrive</font></span><font color="#000000">.MapNetworkDrive(</font><span><font color="#a31515">@"z:\"</font></span><font color="#000000">, </font><span><font color="#a31515">@"\\server\sites\DefaultCollection\Project\TestMatrices"</font></span></font><font color="#000000" size="1"><font size="2">);<br>}</font>	</font></font>

Here I am mapping the z: to the root of the test document library on the Sharepoint portal for my Project. Once this is done, the [DataSource] attribute can just reference any spreadsheet from within this folder:

    <font face="Consolas"><font size="2"><font color="#000000">[</font><span><font color="#2b91af">TestMethod</font></span></font></font><font face="Consolas"><font size="2"><font color="#000000">()]<br>[</font><span><font color="#2b91af">Description</font></span><font color="#000000">(</font><span><font color="#a31515">"Testing database component to add error information"</font></span><font color="#000000">)]</font></font></font><font face="Consolas"><font color="#000000"><br><font size="2">[</font></font><font size="2"><span><font color="#2b91af">DataSource</font></span><font color="#000000">(</font><span><font color="#a31515">"System.Data.Odbc"</font></span><font color="#000000">, </font><span><font color="#a31515">@"Dsn=Excel Files;dbq=z:\Error\ULP_Error_ErrorManager_Add.xlsx;defaultdir=.;driverid=1046;maxbuffersize=2048;pagetimeout=5"</font></span><font color="#000000">, </font><span><font color="#a31515">"ErrorAdd$"</font></span><font color="#000000">, </font><span><font color="#2b91af">DataAccessMethod</font></span></font></font><font face="Consolas"><font size="2"><font color="#000000">.Sequential)]<br></font><span><font color="#0000ff">public</font></span><font color="#000000">&nbsp;</font><span><font color="#0000ff">void</font></span></font><font color="#000000" size="2"> AddErrorTest()<br>{</font></font>

    <font color="#000000" size="2" face="Consolas"> ...</font>

    <font color="#000000" size="2" face="Consolas">}</font>

    <font face="Arial">There’s one other gotcha that you might see – the test run on the build fails saying that the data source could not be opened. In this case, log into your build server as your build service (tfsbuild or whatever identity you’re using) and open any Excel file – make sure that you are not being prompted for your name and initials (a popup that appears the very first time you use Excel). This will break the build until you’ve dismissed this dialogue once.</font>

    <font face="Arial">A last aside: when trying to set up the document library for the spreadsheets, we could not get the Explorer View working in Sharepoint – we just got a “Web page could not be opened”. After using Fiddler, I could see an HTTP 405 error. Googling a bit, I found that in order for Sharepoint’s own WebDAV to work (that’s what the Sharepoint Explorer View uses), you need to uninstall IIS’s WebDAV feature! Go figure…</font>

    <font face="Arial">Happy testing!</font>

