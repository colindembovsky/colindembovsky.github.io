---
layout: post
title: WpfCell – The Key to Coded UI Automation of DataGrids
date: '2011-11-04 17:52:00'
tags:
- testing
---

As a follow on from my [post](http://colinsalmcorner.blogspot.com/2011/11/genericautomationpeer-helping-coded-ui.html) yesterday about some of the intricacies of using Coded UI testing on WPF applications, I wanted to give some details about using the WpfCell class to automate validation and updates into DataGrids.

**The Problem:** When you bind data to a datagrid, the test recorder often complains that the rows (and / or) cells don’t have any “good identification property”. This prevents both validating cell contents as well as recording actions (such as updates) onto the cell.

The reason for this is that the cells don’t have any good automation properties – they all look the same as far as the UIMap is concerned. If you have a DataGrid with a cell that has the text “135” in it, and you find the control using the coded UI Test builder, you’ll see the Name of the cell is “135”. But what if the data isn’t always 135? What if the cell value that displays is dependent on the current record? The framework won’t be able to find that cell and the test will fail.

<!--kg-card-begin: html-->[![image](http://lh5.ggpht.com/-Ja3SyJ25esY/TrOnxFuRPqI/AAAAAAAAAUw/uyUTQK2EA-U/image_thumb%25255B1%25255D.png?imgmax=800 "image")](http://lh4.ggpht.com/-NDzCm1n52lM/TrOnvyA17wI/AAAAAAAAAUo/hYcDlxTMcqI/s1600-h/image%25255B3%25255D.png)<!--kg-card-end: html-->
## The Solution: WpfCell

Using WpfCell (from the [Microsoft.VisualStudio.TestTools.UITesting.WpfControls](http://msdn.microsoft.com/en-us/library/microsoft.visualstudio.testtools.uitesting.wpfcontrols.aspx) namespace) lets us access cells logically (instead of visually). Add the following code to the UIMap.cs file in your coded UI test project:

    internal void ValidateCell(int row, int column, string value)<br>{<br> var cell = FindCell(row, column);<br> Assert.AreEqual(value, cell.Value);<br>}<br><br>internal void UpdateCell(int row, int column, string value)<br>{<br> var cell = FindCell(row, column);<br> cell.Value = value;<br> Keyboard.SendKeys("\t");<br>}<br><br>private WpfCell FindCell(int row, int column)<br>{<br> var cell = new WpfCell(DemoWindow.UIItem.Table);<br> cell.SearchProperties.Add(WpfCell.PropertyNames.RowIndex, row.ToString());<br> cell.SearchProperties.Add(WpfCell.PropertyNames.ColumnIndex, column.ToString());<br> return cell;<br>}<br>

You’ll have to change the Table that is used in the FindCell method – you’ll find the table class generated into the UIMap if you look at the controls (you may have to record a click in the table or something to get this control into the map). The only bit that was interesting here was the Keyboard.SendKeys(“\t”) which presses tab – this actually confirms the update of the value into the cell (otherwise the cell is still “dirty” and the update won’t actually have happened).

Now from within your coded UI Test, you can add code like this:

    UIMap.ValidateCell(0, 1, "135");<br>UIMap.UpdateCell(0, 1, "140");<br>UIMap.ValidateCell(0, 1, "140");<br>

Happy (DataGrid) testing!

