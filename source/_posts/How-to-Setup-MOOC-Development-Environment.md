---
title: How to Setup MOOC OOP Java Development Environment?
date: 2019-02-15 13:24:22
tags: MOOC
categories: Tips
keywords: MOOC, OOP, Java, NetBeans, Intellij IDEA, Development Environment
---

MOOC by University of Helsenki was said to be good and was actually very good. If you are a Java beginner who is searching for well organized exercises, it won't disappoint you. Object-Oriented Programming with Java [Part I](https://materiaalit.github.io/2013-oo-programming/part1/week-1/) and [Part II](https://materiaalit.github.io/2013-oo-programming/part2/week-7/). Register an account at [Test My Code](https://tmc.mooc.fi/user/new). Set up development environment. Finish all the exercises there. That's all. 

![](https://www.dropbox.com/s/nem2jk0x8lldftv/dev-env.jpg?dl=1)<!-- more -->

However, many beginners would have difficulty in setting up development environment, and there are also some things you need to pay attention.

### IDE Selection
I would highly recommend you install [NetBeans 8.2 Java SE](https://netbeans.org/downloads/8.2/) with TMC plugin for Part I exercises. You may also install [TMC-Bundle version](http://update.testmycode.net/installers/tmc-netbeans_org_mooc/tmc-netbeans_org_mooc_tmcbeans-windows.exe). For Part II exercises I suggest you using [Intellij IDEA Community Edition](https://www.jetbrains.com/idea/download/#section=windows).

You need to install JDK first before NetBeans, it's recommended to choose [JDK8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) or lower version for NetBeans installation. For IDEA, it's not needed.

For IDEA, remember to set language level to 1.6 or lower in project structure settings, as the TMC server side compile your code as 1.6. Or you would passed all unit tests locally but failed at server side if you use some features not avaiable in 1.6.

![](https://www.dropbox.com/s/17ow99boghx0bx6/javac-level-6.jpg?dl=1)

Check the NetBeans project file and you would know why.
![](https://www.dropbox.com/s/gpr5a7t3qntiuqz/tmc-javac-source.jpg?dl=1)

Anytime you failed at server side, remember to check error details in submission page.

![](https://www.dropbox.com/s/hd91qe7a6z0nvqr/tmc-server-error.jpg?dl=1)

### Using Chocalatery

Run `cmd.exe` as administrator and copy paste commands in the below, it will automatically install NetBeans 8.2 Java SE and IDEA Community edition.

```cmd
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"

cinst netbeans-jse intellijidea-community -y
```

Follow [instructions of plugin install](https://github.com/UniversityHelsinkiTKTL/tmc-plugin-installation-guide/blob/master/TmcPluginInstallationGuide.md#installing-tmc-plugin-to-exsisting-netbeans-installation-do-this-in-paja) would do the job.

Since package `netbeans-jse` is dependent on `jdk8`, it will also install jdk8 automatically if it's not availalble.

### Attention!
Your NetBeans might not work as expected due to selection of JDK version not supported. In that case, you need to edit configuration file and change JDK home for NetBeans manually.

![](https://www.dropbox.com/s/78nepyips1y113i/netbeans-jdk-home.jpg?dl=1)

You can access all projects in NetBeans at the same time and work on multiple projects easily, however in IDEA you can do one exercise at one time, and need to switch project then, as Part I exercises are usually small, it's really a little bit painful to switch projects too frequently, you know, IDEA is awesome but a little bit slow and heavyweight.

Make sure you pass all local tests first, but this doesn't mean you will pass server side tests. For example, if you use `System.exit(0)` in your `main` method, you would fail at Server side - one of my student uses this and is very confusing why server side failed.

Make sure your output is exactly same as expected, it's very strict, even some words are very simple, don't type them directly, always Copy+Paste them to avoid spelling mistakes.

You can Click `Run -> Test Project`, or right click project and choose `Test`, or click the TMC button `Run tests locally` to perform local tests.

I experienced Copy+Paste issue in NetBeans sometimes, the quickest workaround to resolve it is restarting NetBeans, you may find it's necessary sometimes.

<iframe src="//rcm-na.amazon-adsystem.com/e/cm?o=1&p=48&l=ur1&category=books&banner=0HX1M2P8DDZ20D689R82&f=ifr&linkID=61f529dfb8b35107c92efc29a2c3c8dc&t=javaneversleep-20&tracking_id=javaneversleep-20" width="728" height="90" scrolling="no" border="0" marginwidth="0" style="border:none;" frameborder="0"></iframe>

