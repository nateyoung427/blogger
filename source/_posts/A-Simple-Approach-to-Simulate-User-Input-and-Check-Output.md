---
title: A Simple Approach to Simulate User Input and Check Output
date: 2019-02-07 16:53:12
tags: Tips
categories: Tips
keywords: MOOC, Java, System.in, System.out, Mock, Unit Test
---

Recently some of my students asked me about the mechanism of unit test provided by MOOC from University of Helsinki, I checked their implementation and think it would be helpful for beginners to understand what happened actually, so this little article was posted.

![](https://www.dropbox.com/s/5urgnayz9nch7ca/kick.jpg?dl=1)<!-- more -->

We will use the project ["Airport"](https://materiaalit.github.io/2013-oo-programming/part2/week-7/) as an example, it's the last assignment in the first week of OOP2.

We focus on test only, so I will skip things regarding how to resolve it. For this exercise, we would execute the `main` method manually each time, input plane id, capacity repeatedly, after sometime we think our code would work, we run local tests so that we can submit to server for online judge and grading.

I've been using this little project as an example on refactoring with the help of protection of unit test. When I input plane id, capacity number, airport code and operation code repeatedly and also painfully, I asked my students, "is this painful or not?".

Obviously all of them answered yes. Then I asked, "will you make this kind of test again and again even it's boring and painful?".

Silence.

From my past experience I know that it's easy to skip these boring tests and we can comfort ourselves, "these code are pretty simple and I cannot make a mistake, it will work and would work, don't worry."

I have painful memories because of such choices, because I've made too many simple and stupid mistakes in the past, so no matter how simple it looks, I would still make test - even it's mannual test, boring and painful.

I added this because unit test cannot replace manual test completely, though it will make manual test easier and more effective.

For the Airport project, if we don't need to input repeatedly each time, and we can capture the output of our program, compared to what is expected, we will get feedback much faster.

```java
String operation = scanner.nextLine();
...
System.out.println("Blahblahblah...");
```

For example, we know exactly if we input `x` first, then it will go the Flight Service part and print the menu choices, if we input `x` for the second time then the program will end the loop and quit, as a result, we will only get output of instructions of Airport Panel and Flight Service.

So let's go to a test case to see what will happen actually.

```java
@Test
public void printsMenusAndExits() throws Throwable {
    String syote = "x\nx\n";
    MockInOut io = new MockInOut(syote);
    suorita(f(syote));

    String[] menuRivit = {
        "Airport panel",
        "[1] Add airplane",
        "[2] Add flight",
        "[x] Exit",
        "Flight service",
        "[1] Print planes",
        "[2] Print flights",
        "[3] Print plane info",
        "[x] Quit"
    };

    String output = io.getOutput();
    String op = output;
    for (String menuRivi : menuRivit) {
        int ind = op.indexOf(menuRivi);
        assertRight(menuRivi, syote, output, ind > -1);
        op = op.substring(ind + 1);
    }
}
```

Above is the 2nd test case, which covers the most simple scenario as we said, input two `x` only.

When we look into the test code, it was splitted into 3 parts:
* Prepare input
* execute `Main.main(args)` method
* Check output to see if it contains all expected lines in sequence

You know that the normal behavior of `scanner.nextLine()` or `scanner.nextInt()`. The program will hang on and wait for user's input, so that the next line of code will be executed. But why here it just runs smoothly without any wait?

Before we go to this part I want to explain briefly regarding the execution of the method, it employs Java Reflection to invoke the method in a way not straightforward, but possible to make more check, for example, the first test case requires that `Main` is a public class, but you would probably find that to pass manual test, you can set `Main` access level to package.

```java
@Test
public void classIsPublic() {
    assertTrue("Class " + klassName + " should be public, so it must be defined as\n" +
        "public class " + klassName + " {...\n}", klass.isPublic());
}
```

Here `klass.isPublic()` is checking whether you set access level as required.

OK. It seems that the class [`MockInOut`](https://github.com/testmycode/edu-test-utils/blob/master/src/main/java/fi/helsinki/cs/tmc/edutestutils/MockInOut.java) makes magic happen, we can check the code to find the idea under the hood. You can access the source code at [GitHub](https://github.com/testmycode/edu-test-utils).

```java
public MockInOut(String input) {
    orig = System.out;
    irig = System.in;

    os = new ByteArrayOutputStream();
    try {
        System.setOut(new PrintStream(os, false, charset.name()));
    } catch (UnsupportedEncodingException ex) {
        throw new RuntimeException(ex);
    }

    is = new ByteArrayInputStream(input.getBytes());
    System.setIn(is);
}
```

You might have been typing `System.out` thousands of times, but did you realize that you can change the `out` silently like above? Here it set both `out` and `in` of System, so that we can get the output completely after execution, and we don't need to input manually this time, because in the statement of`Scanner scanner = new Scanner(System.in);`, the parameter `System.in` is changed silently, so that `scanner.nextLine()` will get prepared input without hang on.

Also the output will not be printed in console, but accumulated into the `ByteArrayOutputStream`, which can be accessed afterwards.

You might be wondering that if we really want to restore the normal behavior of `System.in` and `System.out`, what shall we do?

```java
/**
 * Restores System.in and System.out
 */
public void close() {
    os = null;
    is = null;
    System.setOut(orig);
    System.setIn(irig);
}
```

Basically it saves original `in` and `out`, when a restoration is needed, simply clear the hacked ones and set them back, then all will be as usual again.

You can copy the simple sample code in the below for a quick test.

```java
import java.io.*;
import java.util.*;

class HelloWorld {
	public static void main(String[] args) throws IOException {
        PrintStream orig = System.out;

        ByteArrayOutputStream os = new ByteArrayOutputStream();
        System.setOut(new PrintStream(os, false, "UTF-8"));
        // Here it won't print but just accumulate
        for (int i = 0; i < 100; i++) {
            System.out.println("Hello World");
        }

        System.setOut(orig);
        // Print 100 lines of "Hello World" here since out was restored
        System.out.println(os.toString("UTF-8"));

        InputStream is = System.in;
        System.setIn(new ByteArrayInputStream("x\nx\n".getBytes()));
        Scanner scanner = new Scanner(System.in);
        // Without hang on
        System.out.println(scanner.nextLine());
        System.out.println(scanner.nextLine());
        try {
            // There are only two lines provided, so here will fail
            System.out.println(scanner.nextLine());
        } catch (NoSuchElementException e) {
            e.printStackTrace();
        }

        System.setIn(is);
        scanner = new Scanner(System.in);
        // Hang on here since `in` was restored
        System.out.println(scanner.nextLine());
	}
}
```

Actually, inject and replace is a frequently used method to decouple dependencies for unit tests, which is quite useful to focus on your code only. There are more advanced and complex approaches to do this, but here we just want to explain a simple approach that "hack" `in` and `out` so that you can focus on your code, rather than the `in` and `out`.

For some legacy projects, this method might be critical for refactoring, as there are too many heavy dependencies make test really hard! There is a very good book covering this topic, which I read 8 years ago and found it's very useful.

<a target="blank" href="https://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052/ref=as_li_ss_il?ie=UTF8&qid=1549590973&sr=8-1&keywords=working+with+legacy+code&linkCode=li3&tag=javaneversleep-20&linkId=49a9545f458b5002d5b0b74e3e6a1cb2&language=en_US" target="_blank"><img border="0" src="//ws-na.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=0131177052&Format=_SL250_&ID=AsinImage&MarketPlace=US&ServiceVersion=20070822&WS=1&tag=javaneversleep-20&language=en_US" ></a><img src="https://ir-na.amazon-adsystem.com/e/ir?t=javaneversleep-20&language=en_US&l=li3&o=1&a=0131177052" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

<iframe src="//rcm-na.amazon-adsystem.com/e/cm?o=1&p=48&l=ur1&category=prime_student&banner=0MBMTG021FA4HVT1NWG2&f=ifr&linkID=a65bd167cffd233cd211f56ff82069c2&t=javaneversleep-20&tracking_id=javaneversleep-20" width="728" height="90" scrolling="no" border="0" marginwidth="0" style="border:none;" frameborder="0"></iframe>

