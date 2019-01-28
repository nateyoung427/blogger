---
title: 'Fibonacci and BigInteger: Secret Under the Hood'
date: 2019-01-20 22:53:17
tags: Fibonacci
categories: Algorithm
keywords: Fibonacci, BigInteger, Java, Never Sleep
---

No matter what programming language you are learning, Fibonacci numbers calculation is just too classic to skip. It seems very easy to implement, but also can be deep enough to catch up with, especially when the number is super big. Have you thought that ever before?

![](https://www.dropbox.com/s/q88dhrsyh1q4q4i/rabbit.jpg?dl=1)<!-- more -->

It can be super easy to implement using recursion in just one line of code, even using Java.😅

```Java
int fib(int n) {
    return n <= 2 ? 1 : (fib(n - 1) + fib(n - 2));
}
```

And to make you understand the basic optimization idea, monetization mechanism will be introduced via an integer array or a HashMap, but the idea is same: calculate once and only once, so that those poor rabbits would feel life much easier.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/7/7a/FibonacciRabbit.svg/640px-FibonacciRabbit.svg.png)

Add just one more line of code and thanks to the Lambdas feature provided since JDK8, life is much easier nowadays.

```Java
private static Map<Integer, Integer> CACHE = new HashMap<>();

int fib(int n) {
    return CACHE.computeIfAbsent(n, k -> (k <= 2 ? 1 : (fib(k - 1) + fib(k - 2))));
}
```

There is always someone scared of the famous StackOverflowError, they would tell you not to use recursion but do an iterative way, like this:

```Java
int fib(int n) {
    if (n <= 2) return 1;

    int[] cache = new int[n];
    cache[0] = cache[1] = 1;
    for (int i = 2; i < n; i++) {
        cache[i] = cache[i - 1] + cache[i - 2];
    }
    return cache[n - 1];
}
```

Obviously, the integer array can be avoided, you can just keep three values and update them till the end, but that's not the problem we want to discuss here. You would have probably found that it's very easy for the result to be much bigger than the upper limit of int, long, double. For example, if we input n as 2048, what will be the exact result then?

454153044374378942504557144629068920270090826129364442895118239027897145250928343568434971 803477173043320774207501029966396250064078380187973638077418159157949680694899576625922604 895968605634843621876639428348247300097930657521757592440815188064651826480022197557589955 655164820646173515138267042115173436029259905997102292769397103720814141099147144935820441 85153918055170241694035610145547104337536614028338983073680262684101

It's 4.54 * 10^427. As for Java, for this kind of problem, you have no other choice but [BigInteger](https://docs.oracle.com/javase/8/docs/api/java/math/BigInteger.html), and it's very easy to
use.

```Java
private static Map<Integer, BigInteger> CACHE = new HashMap<>();

BigInteger fib(int n) {
    return CACHE.computeIfAbsent(n, k -> (k <= 2 ? BigInteger.ONE : (fib(k - 1).add(fib(k - 2)))));
}
```

Congratulations! You will get a BOMB exploding like this:
```Java
Exception in thread "main" java.lang.StackOverflowError
    at java.util.HashMap.hash(HashMap.java:339)
    at java.util.HashMap.computeIfAbsent(HashMap.java:1099)
```

You may use `-Xss4m` to shut it up, or resolve it in an iterative way a little bit optimized, wherein we won't waste space to build an array. Yes, it's right, recursion is easy to think of and implement, but it's just too close to StackOverflowError.

```Java
BigInteger fib(int n) {
      if (n <= 2) return BigInteger.ONE;

      BigInteger prev = BigInteger.ONE, prevOfPrev = BigInteger.ONE;
      BigInteger curr = null;
      for (int i = 2; i < n; i++) {
            curr = prev.add(prevOfPrev);
            prevOfPrev = prev;
            prev = curr;
      }
      return curr;
}
```

BigInteger, what's the secret behind it that it can represent a number beyond the upper limit of all primitive data types of Java?

The idea is still simple, but the implementation can be complex, especially when highly optimized is a must. [bigint](https://github.com/tbuktu/bigint) in Github would give you more information on how Timothy Buktu approaches and optimizes it.

There is a famous quote in The Lord of The Rings by Sam, "I can't carry it for you, but I can carry you", as for BigInteger, it seems should be "I can't carry it for you, and I can't carry you, but I can carry a segment of you."😅

Yes, the whole number is too big, but if we split it into many parts, then we will find each part might be within the upper limit, if we store each of them in an array, and perform the calculation in binary level, also handle the carrying part well, then a container to store a big number will be possible.

Pretty cool and smart, right? But what exactly is under the hood? Let's check it out in the next post.

_**TO BE CONTINUED**_

<script type="text/javascript">
amzn_assoc_placement = "adunit0";
amzn_assoc_search_bar = "false";
amzn_assoc_tracking_id = "oldyoungboy-20";
amzn_assoc_ad_mode = "manual";
amzn_assoc_ad_type = "smart";
amzn_assoc_marketplace = "amazon";
amzn_assoc_region = "US";
amzn_assoc_title = "";
amzn_assoc_linkid = "21ce171baf5d871f0872d552bc2cbace";
amzn_assoc_asins = "0805063056,B0015DWM2K,0767908163,1590787528";
</script>
<script src="//z-na.amazon-adsystem.com/widgets/onejs?MarketPlace=US"></script>
