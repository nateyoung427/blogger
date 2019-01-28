---
title: 'Refactoring 2nd Edition Available Now! A Masterpiece by Martin Fowler You Shall Never Miss!'
date: 2019-01-18 12:06:24
tags: Refactoring
categories: Great Books
keywords: Refactoring,2nd Edition,Martin Fowler
---

"Refactoring: Improving the Design of Existing Code", the 2nd edition is available now! I read the first edition in 2009 for the first time. It significantly shaped my coding habit, which also made me promoted as the team leader in less than one year - I will tell you why later. After nearly 20 years this book was revised, enhanced, refactored and released. Trust me, you **SHALL NEVER** miss this masterpiece.

<a href="https://amzn.to/2QviuLh"><img src="https://www.dropbox.com/s/dbrkjhh7toukisc/refact2.jpg?dl=1" /></a><!-- more -->

Martin Fowler introduced a term called "Bad Smells" in this book and provides corresponding refactorings to resolve them. The methodology of unit test is a must for successful refactoring, as many programmers misunderstood at this part. They knew bad code smells and determined to remove them and they kept changing, but they didn't perform unit test to make this process reliable and trustable. They usually gave up finally, and the complex, messy legacy code laughed at them in pride loudly.

In my first 3 years experience as a Java Software Engineer, I maintained a code base existing at least 3 years. It's a mystery and a huge mess. You will never know how many new bugs will be introduced after you fix a bug. And it's a nightmare to add some new features - I still remember one of the QA reported to his leader that he even cried when he found that the module is totally untestable - one bug fixed and another new pop up! Each step forward is more difficult than climbing the Himalayas. I don't know whether he wants to kill me or not at that time. It's lucky that I am still alive.

Fortunately, I found the 1st edition of Refactoring in bookshelf and started to read and practice. I applied several methods in the book, such as introduce parameter object, extract method, rename, remove middle class, extract super class and etc. However, I did this in spare time as other colleagues told me such change can be very dangerous, "at least the old code can run, things are not that bad". But as the maintainer, I must say it's too bad to bear any more. My manager is also suspicious and concern a lot about the time would be invested, "new features, rather than reorganization of existing code, are more valuable". Totally agreed! Then I continued in spare time. Talk is cheap, I will show them the result.

I finished the first prototype after two weeks and presented to my manager. As expected, he was very happy to payback such technique debt and he even told me to share this among the team. With the protection of 60+ unit tests and necessary functional test, the finished refactoring made me, QA, project manager, and product manager all super happy. This is not the end. Adding new features or enhancing existing features became much easier than I expected, as after refactoring, the code was well organized and the re-usage rate was pretty good, you just need to change one place in a short time and all done. Previously, thanks to the Copy-Paste code style you never know how many other places were not touched and covered. QA also gave feedback that this module became much stabler than before, which means their test workload is reduced greatly. One of my colleagues was very surprised to see that this module reborn to be a clean, beautiful and easy to maintain code base, "how can this be, man?", he asked.

After my team leader left for IBM, I was immediately promoted as the new team leader. Thanks to Refactoring! However, I would warn you, only those want to make a difference, write better code will be motivated to refactoring and perform unit tests, as this will never be easy. _**Patience, carefulness, boldness**_ are all required to achieve the goal and reach the promise land.

Story time over, let's go back to the topic. Martin Fowler introduced 15 completely new refactorings in 2nd edition, which I'd like to have a try immediately, who knows which role I will be promoted this time?

```
Combine Functions into Class
Combine Functions into Transform
Move Statements into Function
Move Statements to Callers
Remove Dead Code
Rename Field
Rename Variable
Replace Command with Function
Replace Derived Variable with Query
Replace Inline Code with Function Call
Replace Loop with Pipeline
Replace Query with Parameter
Replace Subclass with Delegate
Return Modified Value
Split Phase
```

There are also some other books I once read under this topic and maybe they will be helpful to you, too. I listed them here also. But if you can just pick up one, then it shall always be _**"Refactoring: Improving the Design of Existing Code"**_. Remember, the 2nd Edition this time.

<script type="text/javascript">
amzn_assoc_placement = "adunit0";
amzn_assoc_search_bar = "false";
amzn_assoc_tracking_id = "oldyoungboy-20";
amzn_assoc_ad_mode = "manual";
amzn_assoc_ad_type = "smart";
amzn_assoc_marketplace = "amazon";
amzn_assoc_region = "US";
amzn_assoc_title = "";
amzn_assoc_linkid = "67450530ae19c250a2e7773d9911f72b";
amzn_assoc_asins = "0134757599,0131177052,0132350882,020161622X";
</script>
<script src="//z-na.amazon-adsystem.com/widgets/onejs?MarketPlace=US"></script>