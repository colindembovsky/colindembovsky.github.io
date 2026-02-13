---
layout: post
title: 'Lab Management: Use Database Backups During Deployment for Repeatable Automated
  Tests'
date: '2011-06-17 23:02:00'
tags:
- labmanagement
---

I was at a customer recently who have a small test database – around 120MB in size. I had encouraged them to use the [DB Professional tools in VS 2010](http://msdn.microsoft.com/en-us/library/aa833404.aspx) to track their schema – that way they could deploy the database (as well as the latest schema) and some test data (through INSERT statements or even better, [data generation plans](http://msdn.microsoft.com/en-us/library/aa833267.aspx)). However, their developers didn’t have the time or skills (yet) to do this.

So we initially created the lab with a dependency on a database outside the Lab environment – an _external_ database if you will. However, we were using Network Isolation and had numerous instances of the Lab environments – so contention on the external database was a risk.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-SR3qj05dN4w/Tftec_9ydBI/AAAAAAAAAQ4/3pY5gilG5aE/image_thumb%25255B2%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-qV6WpXmPpRo/TftecOdQt5I/AAAAAAAAAQ0/daxSWiL18So/s1600-h/image%25255B6%25255D.png)<!--kg-card-end: html-->

_The Lab with the External DB Dependency_

I suggested that we install SQLExpress within the Lab environment (since we didn’t have a database VM in the environment, we just installed it on the Client VM where the tests are running). We could have restored a backup of the external database into the Lab as part of the “Clean” snapshot (allowing the Lab to start with a known database), but this would mean any changes to the database schema or lookup data would require someone re-create the snapshot.

So we came up with another solution: we created a shared folder on the network and placed a backup of the database into this folder. Then in the deployment section of the Lab Workflow, we added a script to copy the backup to a folder on the Lab machine (in this case, c:\data). The last step is to use sqlcmd to restore the database as follows (paths and dbnames depend on your environment, of course):

<!--kg-card-begin: html--><font face="Courier New">cmd /c sqlcmd -S .\SqlExpress -Q "RESTORE DATABASE Demo FROM Disk='c:\data\Lab DB.bak' WITH MOVE 'DataLogicalName' TO 'c:\data\Lab DB.mdf', MOVE 'LogLogicalName' TO 'c:\data\Lab DB.ldf'"</font><!--kg-card-end: html-->

where DataLogicalName and LogLogicalName are the logical names of the data and log files within the backup.

This approach allows you to “deploy” schema changes by refreshing the backup on the shared location. If you add a new table or something that the code requires, simply backup the new database to the shared location and you’re done.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-cncp3DtmnD8/TfteeSMoFXI/AAAAAAAAARA/RxUOv7S1tgw/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-8qeXFB6Jeew/TftedTCeRSI/AAAAAAAAAQ8/iFeSMAXtRlc/s1600-h/image%25255B9%25255D.png)<!--kg-card-end: html-->

_The Lab at the “Clean” Snapshot_

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-alUXc0p651E/Tftef-LuRUI/AAAAAAAAARI/JlzqfxT61UM/image_thumb%25255B4%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-LG-NcoKoKa8/TftefPEiXVI/AAAAAAAAARE/ZomMBSE4gbQ/s1600-h/image%25255B12%25255D.png)<!--kg-card-end: html-->

_The Lab after deployment_

Happy testing!

