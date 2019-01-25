---
title: 'Behind-the-Scenes Secrets of Jsoup V: Tips & Tricks of Optimization'
date: 2019-01-16 09:02:51
tags:
- Jsoup
categories: Code Review
keywords: Jsoup, Code Review, Optimization, Tips and Tricks, Performance
---

We have done things right, now it's time to do things faster. We would keep [Donald Knuth](https://en.wikipedia.org/wiki/Donald_Knuth)'s warning in mind, "We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil".

![](../images/racing-horse.jpg)<!-- more -->

According to Jonathan Hedley, he uses [YourKit Java Profiler](https://www.yourkit.com/java/profiler/) to measure memory usage and find the performance hot point. Using statistic result of such kind of tools is crucial for the success of optimizations, it will prevent you spend time just wondering and making useless tunings, which doesn't improve performance but also make your code unnecessarily complex and hard to maintain. Jonathan also talked about this in the ["Colophon"](https://jsoup.org/colophon).

We will list some tips and tricks used in Jsoup, they are randomly ordered currently, would be re-organized in the future.

### 1. Padding for Indent
```java
// memoised padding up to 21, from "", " ", "  " to "                   "
static final String[] padding = {......};

public static String padding(int width) {
    if (width < 0)
        throw new IllegalArgumentException("width must be > 0");

    if (width < padding.length)
        return padding[width];
    char[] out = new char[width];
    for (int i = 0; i < width; i++)
        out[i] = ' ';
    return String.valueOf(out);
}

protected void indent(Appendable accum, int depth, Document.OutputSettings out) throws IOException {
    accum.append('\n').append(StringUtil.padding(depth * out.indentAmount()));
}
```

Pretty smart, right? It maintains a cache of different lengths of paddings which would cover 80% of the cases - which I assume would be based on the author's experience and statistic.

### 2. Has Class Or Not?

[`Element#hasClass`](https://github.com/jhy/jsoup/blob/master/src/main/java/org/jsoup/nodes/Element.java#L1270) was marked as **performance sensitive**, for example, we want to check whether `<div class="logged-in env-production intent-mouse">` has class `production`, split class by whitespace to an array then loop and search would work, but in a deep traverse this would be in-efficiency. Jsoup introduces **Early Exit** here first, by compare length with target class name to avoid unnecessary scan and search, which will also be beneficial. Then it uses a pointer detecting whitespace and performs regionMatches - honestly speaking this is the first time I got to know method `String#regionMatches`🙈😅

```java
public boolean hasClass(String className) {
    final String classAttr = attributes().getIgnoreCase("class");
    final int len = classAttr.length();
    final int wantLen = className.length();

    if (len == 0 || len < wantLen) {
        return false;
    }

    // if both lengths are equal, only need compare the className with the attribute
    if (len == wantLen) {
        return className.equalsIgnoreCase(classAttr);
    }

    // otherwise, scan for whitespace and compare regions (with no string or arraylist allocations)
    boolean inClass = false;
    int start = 0;
    for (int i = 0; i < len; i++) {
        if (Character.isWhitespace(classAttr.charAt(i))) {
            if (inClass) {
                // white space ends a class name, compare it with the requested one, ignore case
                if (i - start == wantLen && classAttr.regionMatches(true, start, className, 0, wantLen)) {
                    return true;
                }
                inClass = false;
            }
        } else {
            if (!inClass) {
                // we're in a class name : keep the start of the substring
                inClass = true;
                start = i;
            }
        }
    }

    // check the last entry
    if (inClass && len - start == wantLen) {
        return classAttr.regionMatches(true, start, className, 0, wantLen);
    }

    return false;
}
```

### 3. Tag Name In or Not?

As we analyzed in previous articles, `HtmlTreeBuilderState` will validate nest correctness by checking whether tag name in a certain collection or not. We can compare the implementation before and after _1.7.3_ to have a check.

```java
// 1.7.2
} else if (StringUtil.in(name, "base", "basefont", "bgsound", "command", "link", "meta", "noframes", "script", "style", "title")) {
    return tb.process(t, InHead);
}

// 1.7.3
static final String[] InBodyStartToHead = new String[]{"base", "basefont", "bgsound", "command", "link", "meta", "noframes", "script", "style", "title"};
...
} else if (StringUtil.inSorted(name, Constants.InBodyStartToHead)) {
    return tb.process(t, InHead);
}        
```

According to the comment written by the author, "A little harder to read here, but causes less GC than dynamic varargs. Was contributing around 10% of parse GC load. Must make sure these are sorted, as used in findSorted". Simply using `static final` constant array, also make them sorted so that binary search will also improve from O(n) to O(log(n)), the cost–performance ratio is pretty good here.

However, "MUST update HtmlTreebuilderStateTest if more arrays added" is not a good way to synchronize IMHO, rather than Copy & Paste I would use reflection to retrieve those constants. You may find my proposal in Pull Request [#1157: "Simplify state sorting status unit test - avoid duplicated code in HtmlTreeBuilderStateTest.java"](https://github.com/jhy/jsoup/pull/1157).

### 4. Flyweight Pattern

Do you know the trick of `Integer.valueOf(i)`? It maintains a `IntegerCache` cache from -128 to 127 or higher if configured(`java.lang.Integer.IntegerCache.high`), as a result, `==` and `equals` result will be different when the value is located in a different range(a classic Java interview question?). This is an example of [Flyweight Pattern](https://en.wikipedia.org/wiki/Flyweight_pattern) actually. As for Jsoup, applying this pattern will also reduce object created times and gain benefit to performance.

```java
/**
 * Caches short strings, as a flywheel pattern, to reduce GC load. Just for this doc, to prevent leaks.
 * <p />
 * Simplistic, and on hash collisions just falls back to creating a new string, vs a full HashMap with Entry list.
 * That saves both having to create objects as hash keys, and running through the entry list, at the expense of
 * some more duplicates.
 */
private static String cacheString(final char[] charBuf, final String[] stringCache, final int start, final int count) {
    // limit (no cache):
    if (count > maxStringCacheLen)
        return new String(charBuf, start, count);
    if (count < 1)
        return "";

    // calculate hash:
    int hash = 0;
    int offset = start;
    for (int i = 0; i < count; i++) {
        hash = 31 * hash + charBuf[offset++];
    }

    // get from cache
    final int index = hash & stringCache.length - 1;
    String cached = stringCache[index];

    if (cached == null) { // miss, add
        cached = new String(charBuf, start, count);
        stringCache[index] = cached;
    } else { // hashcode hit, check equality
        if (rangeEquals(charBuf, start, count, cached)) { // hit
            return cached;
        } else { // hashcode conflict
            cached = new String(charBuf, start, count);
            stringCache[index] = cached; // update the cache, as recently used strings are more likely to show up again
        }
    }
    return cached;
}
```

There is also another scenario to minimize new StringBuilder GCs using the same idea.

```java
private static final Stack<StringBuilder> builders = new Stack<>();

/**
 * Maintains cached StringBuilders in a flyweight pattern, to minimize new StringBuilder GCs. The StringBuilder is
 * prevented from growing too large.
 * <p>
 * Care must be taken to release the builder once its work has been completed, with {@see #releaseBuilder}
*/
public static StringBuilder borrowBuilder() {
    synchronized (builders) {
        return builders.empty() ?
            new StringBuilder(MaxCachedBuilderSize) :
            builders.pop();
    }
}
```

Actually, [`CharacterReader`](https://github.com/jhy/jsoup/blob/master/src/main/java/org/jsoup/parser/CharacterReader.java) and [`StringUtil`](https://github.com/jhy/jsoup/blob/master/src/main/java/org/jsoup/internal/StringUtil.java) are worthy to digest more and more as there are many useful tips and tricks that will inspire you.

### 5. Other Improvement Methods

* Use RandomAccessFile to read files that improved file read time by 2x. Check [#248](https://github.com/jhy/jsoup/issues/248) for more details
* Node hierarchy refactoring. Check [#911](https://github.com/jhy/jsoup/issues/911) for more details
* "Improvements largely from re-ordering the HtmlTreeBuilder methods based on analysis of various websites" - I list this one here because it's very practical. Deeper understanding and observation of how the code will run will also give you some insights
* Call `list.toArray(0)` rather than `list.toArray(list.size())` - this has been used in certain open source projects such as [h2database](https://github.com/h2database/h2database/issues/311), so I also proposed this in another Pull Request [#1158](https://github.com/jhy/jsoup/pull/1158)

### 6. The Unknowns

Optimization never ends. There are still many tips and tricks I didn't discover at this time. I would appreciate if you can share them to me if you find more inspiring ideas in Jsoup. You may find my contact information in the left sidebar of this website or simply email to `ny83427 at gmail.com`.

**-To Be Continued-**