---
layout: post
title: 'Workspaces: Updating the owner'
date: '2012-01-17 19:19:00'
tags:
- sourcecontrol
---

Recently we changed our internal domain. It was a little bit of a pain, since I had to migrate my setting and preferences. One “snag” I hit was reconnecting to TFS once I had changed my primary login from the old domain to the new domain.

If you change your username (or domain), open Visual Studio and go to the Source Control Explorer, you’ll see that (of course) you don’t have any mappings in your default workspace. If you try to create a mapping to an existing folder on your hard-drive, you’ll get an error stating that the folder is mapped in another workspace:

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-3aTW1F9wLTo/TxU83qyh8sI/AAAAAAAAAVA/_t6o80zFlS8/image_thumb1.png?imgmax=800 "image")](http://lh4.ggpht.com/-lUJI6YV8boc/TxU81Kcm_5I/AAAAAAAAAU4/YSmqTMWiexY/s1600-h/image3.png)<!--kg-card-end: html-->

So you drop into the Visual Studio command prompt and use the tf.exe command to list the workspaces:

<!--kg-card-begin: html--><font face="Courier New">tf workspaces</font><!--kg-card-end: html--><!--kg-card-begin: html-->[![image](http://lh6.ggpht.com/-nx-bTh1nkjc/TxU87lEusaI/AAAAAAAAAVQ/hQGM92PRhIs/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-eJy28msIT_w/TxU85nC-v7I/AAAAAAAAAVI/veXItMsiIrY/s1600-h/image%25255B4%25255D.png)<!--kg-card-end: html-->

However (and this may be a bug, I’m not sure), this only brings back workspaces for the current user (on the new domain). So if you want to see all the workspaces (for any user) you’ll have to enter this command:

<!--kg-card-begin: html--><font face="Courier New">tf workspaces /collection:tpcUrl /owner:*</font><!--kg-card-end: html-->

(Of course tpcUrl is the url of your team project collection).

<!--kg-card-begin: html-->[![image](http://lh4.ggpht.com/-kctwalPO8mg/TxU8_OxuP_I/AAAAAAAAAVg/2_qAvJrWagc/image_thumb%25255B3%25255D.png?imgmax=800 "image")](http://lh6.ggpht.com/-xDCuCF8GPFs/TxU89DyI8bI/AAAAAAAAAVY/Jekvqx2H_fI/s1600-h/image%25255B8%25255D.png)<!--kg-card-end: html-->

Now that you can see all the workspaces, you’ll want to update the owner of the old workspace.

<!--kg-card-begin: html--><font face="Courier New">tf workspace /name:workspacename;workspaceowner /collection:tpcUrl</font><!--kg-card-end: html-->

This launches the dialogue for the workspace properties. So I tried to just update the name in the owner field, but there were pending changes in the workspace, so I got an error message:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-eV3OTwvKX34/TxU9EEbpawI/AAAAAAAAAVw/8P-18lr6jtU/image_thumb%25255B5%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-B5nJv8SHY40/TxU9B6eTWXI/AAAAAAAAAVo/8lKa6aCnehk/s1600-h/image%25255B12%25255D.png)<!--kg-card-end: html-->

Back to the console. To see the pending changes in the old workspace change dir to the root of your workspace mapping and run the following command:

<!--kg-card-begin: html--><font face="Courier New">tf status /collection:tpcUrl /workspace:workspacename;workspaceowner</font><!--kg-card-end: html-->

This lists all the pending changes. Now from the list of changes, I could see some that I wanted to keep and others that I didn’t – but I didn’t want to think about sorting the changes out now. So I decided to follow the previous error log’s advice and shelve the pending changes.

To “select” the old workspace from the command prompt, run the tf workspace command:

<!--kg-card-begin: html--><font face="Courier New">tf workspace /name:workspacename;workspaceowner /collection:tpcUrl</font><!--kg-card-end: html-->

When the dialogue opens, change the “Permissions” to Public (if it’s not set already) to make sure that the new user can shelve in the old user’s workspace and press the OK button.

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-InKi0a4e6ac/TxU9ILxZDDI/AAAAAAAAAWA/-d84ixOfjTw/image_thumb%25255B7%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-rWQ-NNl6yE4/TxU9FwnnuWI/AAAAAAAAAV4/MIHVq7_A_Ps/s1600-h/image%25255B16%25255D.png)<!--kg-card-end: html-->

Now you can run the shelve command:

<!--kg-card-begin: html--><font face="Courier New">tf shelve /recursive name;owner *.* /move</font><!--kg-card-end: html-->

(Here name is the name of the shelveset and owner is the domain\username of the new user)

The shelve dialogue opens, so you can review the shelveset. Press Shelve to complete the operation.

Now run the workspace command again, and change the owner:

<!--kg-card-begin: html-->[![image](http://lh3.ggpht.com/-k1O-d3zcd9M/TxU9MP4p0zI/AAAAAAAAAWQ/2E30rPD3WLw/image_thumb%25255B9%25255D.png?imgmax=800 "image")](http://lh3.ggpht.com/-fAllOFTm59s/TxU9J7qW-lI/AAAAAAAAAWI/3D7-zg01PII/s1600-h/image%25255B20%25255D.png)<!--kg-card-end: html-->

(I had to rename the new workspace since it had the same name as the old one before this worked).

Finally, to get the pending changes back, you need to unshelve. Open the Pending Changes window (View-\>Other Windows-\>Pending Changes in VS) and press the Unshelve button. Find the shelveset that you saved the pending changes in and select it. Then hit Unshelve and your pending changes are back.

Now your workspace is updated to the new user, and you can get back to work!

