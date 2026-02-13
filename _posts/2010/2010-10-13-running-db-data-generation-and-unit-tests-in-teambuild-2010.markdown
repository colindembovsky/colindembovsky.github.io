---
layout: post
title: Running DB Data Generation and Unit Tests in TeamBuild 2010
date: '2010-10-13 23:38:00'
tags:
- build
---

Using “Data Dude” (or VS DB Professional) in VS 2010 allows you to bring Agile to your database development. You can import a database schema and add it to source control – so you get linking to work items as well as branching and labelling etc. Furthermore, you can use TeamBuild to build a .schema file, and then use vsdbcmd.exe to deploy schema changes (incrementally) to databases. You can also create unit tests and data generation plans.

Importing the schema and building the schema in TeamBuild is straightforward. However, if you want to add running data generation and unit tests in your build, you have to jump through a few hoops.

Let’s go through an example.

First, we’ll import the schema into a database project using the SQL 2008 Database Wizard project template.

<!--kg-card-begin: html-->[![Schema explorer](http://lh5.ggpht.com/_d41Ixos7YsM/TLXD9-ToJVI/AAAAAAAAALY/EaEyRWt3-OQ/schemaexplorer_thumb1.png?imgmax=800 "Schema explorer")](http://lh4.ggpht.com/_d41Ixos7YsM/TLXD8GfKUPI/AAAAAAAAALU/9AjFDPLyJ9s/s1600-h/schemaexplorer3.png)<!--kg-card-end: html-->

Next, right click the Data Generation Plans folder and add a new Data Generation plan. The details are not important for this example, so just accept the defaults. Press F5 to generate data to the database.

<!--kg-card-begin: html-->[![Data generation plan](http://lh5.ggpht.com/_d41Ixos7YsM/TLXECothJbI/AAAAAAAAALg/dU8o_Ny0K34/datageneration_thumb1.png?imgmax=800 "Data generation plan")](http://lh6.ggpht.com/_d41Ixos7YsM/TLXEANaMFqI/AAAAAAAAALc/g-08J70UYqk/s1600-h/datageneration3.png)<!--kg-card-end: html-->

Now go to the schema view and select a stored proc. Right click and select “Create Unit Tests”.

<!--kg-card-begin: html-->[![Unit test settings](http://lh5.ggpht.com/_d41Ixos7YsM/TLXEHr_9qzI/AAAAAAAAALo/dC0LIUmUOxk/testSettings_thumb1.png?imgmax=800 "Unit test settings")](http://lh6.ggpht.com/_d41Ixos7YsM/TLXEFMowDMI/AAAAAAAAALk/xKzkmQrIlOI/s1600-h/testSettings3.png)<!--kg-card-end: html-->

Create a new test project and fill in some tests. Again, the exact details here are not critical – just make sure you’ve got some tests that pass when you run them from Visual Studio.

<!--kg-card-begin: html-->[![Tests pass locally](http://lh3.ggpht.com/_d41Ixos7YsM/TLXEKzXyI9I/AAAAAAAAALw/7m08cILeasM/testspasslocallyPNG_thumb1.png?imgmax=800 "Tests pass locally")](http://lh5.ggpht.com/_d41Ixos7YsM/TLXEJACDllI/AAAAAAAAALs/19v_YQA8L7g/s1600-h/testspasslocallyPNG3.png)<!--kg-card-end: html-->

Next you’ll have to create a build. Check you solution into source control. Then right click the builds node of the project and create a new build definition. Choose the workspace and the drop location, and leave everything else defaulted. This generates a standard build that will compile the database project and then run the unit tests.

Unfortunately, this build will only partially succeed – the unit tests will fail. This is due to the fact that the paths to the .dbproj and .dgen files are different during a TeamBuild than when we build in VS. So we have to modify the configs.

<!--kg-card-begin: html-->[![Build only partially succeeds](http://lh5.ggpht.com/_d41Ixos7YsM/TLXEQdrasFI/AAAAAAAAAL4/8DAo39beE4Q/buildfailed_thumb1.png?imgmax=800 "Build only partially succeeds")](http://lh4.ggpht.com/_d41Ixos7YsM/TLXENju0bAI/AAAAAAAAAL0/zXiFUsAJkLM/s1600-h/buildfailed3.png)<!--kg-card-end: html--><!--kg-card-begin: html-->[![The deployment path is wrong](http://lh6.ggpht.com/_d41Ixos7YsM/TLXETD1EetI/AAAAAAAAAMA/QOGKNFKmtOQ/pathfailed_thumb1.png?imgmax=800 "The deployment path is wrong")](http://lh3.ggpht.com/_d41Ixos7YsM/TLXERqFXHzI/AAAAAAAAAL8/DHw2_zm2VEo/s1600-h/pathfailed3.png)<!--kg-card-end: html-->

Right click the app.config &nbsp;of the Test Project. Copy and paste it so that you have a copy of the config. Rename this to _buildserver_.dbunittest.config where _buildserver_ is the name of your build server (this could also be the name of the user running the build). The .dbunittest is important since TeamBuild looks for \*.dbunittest.config if we override the default config.

Now remove all the tags except the &nbsp;and its contents. Make sure that the very 1st character of the file is the \<, else TeamBuild won’t read this config correctly. Add “..\Sources\” into the paths to the .dbproj file and .dgen files (just after the ..\..\).

\<DatabaseUnitTesting\>  
 &nbsp; &nbsp;\<DatabaseDeployment DatabaseProjectFileName="..\..\..\Sources\TailspinToys\TailspinToys.dbproj"  
 &nbsp; &nbsp; &nbsp; &nbsp;Configuration="Debug" /\>  
 &nbsp; &nbsp;\<DataGeneration DataGenerationFileName="..\..\..\Sources\TailspinToys\Data Generation Plans\UnitTest.dgen"  
 &nbsp; &nbsp; &nbsp; &nbsp;ClearDatabase="true" /\>  
 &nbsp; &nbsp;\<ExecutionContext Provider="System.Data.SqlClient" ConnectionString="Data Source=TRAIN-TFS;Initial Catalog=TailspinToys;Integrated Security=True;Pooling=False"  
 &nbsp; &nbsp; &nbsp; &nbsp;CommandTimeout="30" /\>  
 &nbsp; &nbsp;\<PrivilegedContext Provider="System.Data.SqlClient" ConnectionString="Data Source=TRAIN-TFS;Initial Catalog=TailspinToys;Integrated Security=True;Pooling=False"  
 &nbsp; &nbsp; &nbsp; &nbsp;CommandTimeout="30" /\>  
DatabaseUnitTesting\>

Open up the app.config in the Test project. Add AllowConfigurationOverride=”true” to the &nbsp;tag. This prompts TeamBuild to look for the config we created in the previous step.

\<DatabaseUnitTesting AllowConfigurationOverride="true"\>

<!--kg-card-begin: html-->[![Modified test project](http://lh5.ggpht.com/_d41Ixos7YsM/TLXEUWQOH3I/AAAAAAAAAMI/KNjDzioKb10/modifiedtestproject_thumb2.png?imgmax=800 "Modified test project")](http://lh6.ggpht.com/_d41Ixos7YsM/TLXEThTmjQI/AAAAAAAAAME/0BQEbrTNHic/s1600-h/modifiedtestproject4.png)<!--kg-card-end: html-->

Finally we need to create a new test settings file. Right click the Solution Items folder and add a new item – select the Test Settings item. You can name this whatever you want to and set whatever you want for your settings. The important bit here is to go to the “Deployment” tag and check the “Enable Deployment”. Then click the “Add” button and browse to the _buildserver_.dbunittest.config file (you’ll have to change the file-type drop-down to \*.\*). Check in your solution.

<!--kg-card-begin: html-->[![deploy settings](http://lh3.ggpht.com/_d41Ixos7YsM/TLXEW5Nu4LI/AAAAAAAAAMQ/j0d8gQIOdWs/deploysettings_thumb2.png?imgmax=800 "deploy settings")](http://lh5.ggpht.com/_d41Ixos7YsM/TLXEVP_jh9I/AAAAAAAAAMM/q_8jj31KEMw/s1600-h/deploysettings4.png)<!--kg-card-end: html-->

Check your solution into source control.

Finally we’ll modify the build. Edit the build definition that you created before. In the Process tab, click the ellipses after the Automated Test Settings to get the test settings dialogue. Set the Test Settings File to be the test settings we just created. If you can’t see the file, it may be that you haven’t checked it into source control – this browser browses source control, not your local drive.

<!--kg-card-begin: html-->[![Configure build test settings](http://lh3.ggpht.com/_d41Ixos7YsM/TLXEZsiXKCI/AAAAAAAAAMY/waoJeA41ilM/configurebuildtestsettings_thumb1.png?imgmax=800 "Configure build test settings")](http://lh4.ggpht.com/_d41Ixos7YsM/TLXEXuI48DI/AAAAAAAAAMU/G4pOLXZ6m7E/s1600-h/configurebuildtestsettings3.png)<!--kg-card-end: html-->

Now queue a build and check that your tests are passing!

<!--kg-card-begin: html--> [![completed](http://lh4.ggpht.com/_d41Ixos7YsM/TLXEd30ndbI/AAAAAAAAAMg/P36cUt6IXI4/completed_thumb1.png?imgmax=800 "completed")](http://lh6.ggpht.com/_d41Ixos7YsM/TLXEbMl8i4I/AAAAAAAAAMc/hxaV4dlmxiY/s1600-h/completed3.png)<!--kg-card-end: html-->