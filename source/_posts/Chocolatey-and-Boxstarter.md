---
title: 'Chocolatey and Boxstarter: Setup Development Environment on Windows Like Never Before'
date: 2018-12-12 21:58:41
tags: Chocolatey
categories: Devops
keywords: jdk download, Chocolatey, Boxstarter, Automation, Software Management
---

How many times have you googled "jdk download windows", then download, click, click, and click? You don't have to be like this even if you are using Windows. [Chocolatey](https://chocolatey.org) and [Boxstarter](https://boxstarter.org/) are gospel for Windows users. Below is just an example: One Command To Install Them All!

```shell
cinst jdk8 jdk11 git intellijidea-community vscode nodejs everything f.lux -y
```
![](../images/trick.jpg)<!-- more -->

Install Chocolatey is super easy. Simply follow the [instructions](https://chocolatey.org/install) will do. Me usually choose the first option "install with cmd.exe(Run as Administrator)", you just need to execute the command in the below, and you will get the Windows version **apt-get**:
```shell
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

Wanna upgrade? `choco upgrade chocolatey`

You may explore _**6161**_ community maintained [packages](https://chocolatey.org/packages) and build your own toolkit scripts. I've found it to be super useful when I need to setup development environment in bunches of different computers in a short time without any mistake. As sometimes I just cannot reproduce some tricky bugs in my own laptop and have to investigate in end user's computer.

You can be even happier using [Boxstarter](https://boxstarter.org/), it's very straightforward and easy to use also.

{% youtube sgzVTG-zIPE %}

Honestly speaking, I didn't dive into these two tools deeply as I found the most frequently used commands would resolve my problem perfectly. If you use Windows as daily development environment, I highly recommend you to automate your setup process using them.

Yeah! Happy Coding! You may also access the [gist](https://gist.github.com/ny83427/4ca8801fb340bb0555e63155a7050ee9) created for this post.