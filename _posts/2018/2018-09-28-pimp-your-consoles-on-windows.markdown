---
layout: post
title: Pimp Your Consoles on Windows
date: '2018-09-28 21:45:35'
tags:
- development
---

1. TOC
{:toc}

I spend a fair amount of time in consoles - specifically PowerShell and Bash (Windows Subsystem for Linux) on my Windows 10 machine. I also work with Git - a lot. So having a cool console that is Git aware is a must. I always recommend [Posh-Git](https://www.powershellgallery.com/packages/posh-git/0.7.1) (a PowerShell prompt that shows you which Git branch you're on as well as the branch status). At Ignite this I saw some zsh consoles in VS Code on Mac machines. So I wondered if I could get my consoles to look as cool. And it's not just about the looks - seeing your context in the console is a productivity booster!

It turns out that other than installing some updated fonts for both PowerShell and Bash, you can get pretty sweet consoles fairly easily.

## Updating Fonts

The fonts that the custom shells use are UTF-8, so you'll need UTF-8 fonts installed. You'll also need so-called "powerline" fonts. Fortunately, there's a simple script you can run to install a whole bunch of cool fonts that will work nicely on your consoles.

Here are the steps for installing the fonts. Open a PowerShell and enter the following commands:

    git clone https://github.com/powerline/fonts.git
    .\install.ps1

This took about 5 minutes on my machine.

## PowerShell

So to pimp out your PowerShell console, you'll need to install a couple modules: [Posh-Git](https://www.powershellgallery.com/packages/posh-git/0.7.1) and [Oh-My-Posh](https://www.powershellgallery.com/packages/oh-my-posh/2.0.225). Run

<!--kg-card-begin: html--><font face="Courier New">Install-Module Posh-Git</font><!--kg-card-end: html-->

and

<!--kg-card-begin: html--><font face="Courier New">Install-Module Oh-My-Posh<font face="Calibri">. Once both modules are installed, you need to edit your <font face="Courier New">$PROFILE</font> (you can run <font face="Courier New">code $PROFILE</font> to quickly open your profile file in VSCode). Add the following lines:</font></font><!--kg-card-end: html-->

    Install-Module posh-git
    Install-Module oh-my-posh
    Set-Theme Paradox

You can of course choose different themes - run

<!--kg-card-begin: html--><font face="Courier New">Get-Theme</font><!--kg-card-end: html-->

to get a list of themes. One last thing to do - set the background color of your PowerShell console to black (I like to make the opacity 90% too).

Now if you cd to a git repo, you'll get a Powerline status. Sweet!

<!--kg-card-begin: html--> [![image](/assets/images/files/f829659d-0185-4c9f-bfb0-8c663b615448.png "image")](/assets/images/files/a1a635c3-cc83-461d-a821-fcbb04a06ec0.png)<!--kg-card-end: html-->
## Bash

You can do the same thing for your Bash console. I like to use [fish shell](https://github.com/fish-shell/fish-shell) so you'll have to install that first. Once you have fish installed, you can install [oh-my-fish](https://github.com/oh-my-fish/oh-my-fish) - a visual package manager for fish (and yes, oh-my-posh is a PowerShell version of oh-my-fish). Once oh-my-fish is installed, use it to install themes. You can install agnoster by running

<!--kg-card-begin: html--><font face="Courier New">omf install agnoster</font><!--kg-card-end: html-->

- I like bobthefish, so I just run

<!--kg-card-begin: html--><font face="Courier New">omf install bobthefish</font><!--kg-card-end: html-->

. Now my bash console is pimped too!

<!--kg-card-begin: html--> [![image](/assets/images/files/1b0ceee7-2ab0-4dfb-a12d-2ba8bcfcd985.png "image")](/assets/images/files/413ad7e5-a037-4fad-9691-41fb8798fef6.png)<!--kg-card-end: html-->
## Solarized Theme

One more change you may want to make: update your console colors to the Solarized theme. To do that, follow the instructions from [this repo](https://github.com/neilpa/cmd-colors-solarized).

## Conclusion

If you're going to work in a console frequently, you may as well work in a pretty one! Oh-My-Fish and Oh-My-Posh let you quickly and easily get great-looking consoles, and Posh-Git adds in Git context awareness. What's not to love?

Happy console-ing!

