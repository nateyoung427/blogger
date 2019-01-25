---
title: 'Behind-the-Scenes Secrets of Jsoup IV: CSS Selector'
date: 2019-01-14 20:40:04
tags:
- Jsoup
categories: Code Review
keywords: Jsoup, Code Review, Parser, Github, CSS Selector
---

Most of the time, you are just consuming the `Document` Jsoup built for you, like `document.select(${selector})`. It would be helpful to go through [W3C CSS Selector Specification](https://www.w3.org/TR/CSS2/selector.html) to understand Jsoup's roadmap. We have talked about Node Traverse before, what `Selector` will actually do is just filtering and collecting while traversing, and the key point here will be parsing and evaluating given queries.

![](../images/make-coffee.jpg)<!-- more -->

### Overview of Package

Package `org.jsoup.select` didn't change much in latest version _1.12.1-SNAPSHOT_, I would reference UML diagram made by [Yihua Huang](https://github.com/code4craft/) directly here:

![](../images/select-uml-Yihua-Huang.jpeg)

Besides `NodeVistor` the interface, there is another similar interface `NodeFilter`, their methods are nearly the same:

```java
public interface NodeFilter {
    /**
     * Filter decision.
     */
    enum FilterResult {
        /** Continue processing the tree */
        CONTINUE,
        /** Skip the child nodes, but do call {@link NodeFilter#tail(Node, int)} next. */
        SKIP_CHILDREN,
        /** Skip the subtree, and do not call {@link NodeFilter#tail(Node, int)}. */
        SKIP_ENTIRELY,
        /** Remove the node and its children */
        REMOVE,
        /** Stop processing */
        STOP
    }

    /**
     * Callback for when a node is first visited.
     */
    FilterResult head(Node node, int depth);

    /**
     * Callback for when a node is last visited, after all of its descendants have been visited.
     */
    FilterResult tail(Node node, int depth);
}
```

There is only one implementation in production code, which is `FirstFinder`, 5 other implementations are all located in test and probably reserved for the future. As for `FirstFinder`, it's easy to guess that it's created in the purpose of optimization. You don't need to collect all the matching elements and return the first one, instead of invoking `select(${selector}).get(0)`, Jsoup introduces another method `selectFirst(${selector})` which will be useful when you just care about the first one - a frequent scenario actually.

### NodeFilter: Another Traverse

I don't want to repeat Node Traverse here again, but it will be helpful to review it in another way - the implementation of filtering. The idea and workflow are totally the same, but a little bit longer. You may compare to [`NodeTraversor#traverse`](https://github.com/jhy/jsoup/blob/master/src/main/java/org/jsoup/select/NodeTraversor.java#L40) for better understanding.

```java
public static FilterResult filter(NodeFilter filter, Node root) {
    Node node = root;
    int depth = 0;

    while (node != null) {
        FilterResult result = filter.head(node, depth);
        if (result == FilterResult.STOP)
            return result;
        // Descend into child nodes:
        if (result == FilterResult.CONTINUE && node.childNodeSize() > 0) {
            node = node.childNode(0);
            ++depth;
            continue;
        }
        // No siblings, move upwards:
        while (node.nextSibling() == null && depth > 0) {
            // 'tail' current node:
            if (result == FilterResult.CONTINUE || result == FilterResult.SKIP_CHILDREN) {
                result = filter.tail(node, depth);
                if (result == FilterResult.STOP)
                    return result;
            }
            Node prev = node; // In case we need to remove it below.
            node = node.parentNode();
            depth--;
            if (result == FilterResult.REMOVE)
                prev.remove(); // Remove AFTER finding parent.
            result = FilterResult.CONTINUE; // Parent was not pruned.
        }
        // 'tail' current node, then proceed with siblings:
        if (result == FilterResult.CONTINUE || result == FilterResult.SKIP_CHILDREN) {
            result = filter.tail(node, depth);
            if (result == FilterResult.STOP)
                return result;
        }
        if (node == root)
            return result;
        Node prev = node; // In case we need to remove it below.
        node = node.nextSibling();
        if (result == FilterResult.REMOVE)
            prev.remove(); // Remove AFTER finding sibling.
    }
    // root == null?
    return FilterResult.CONTINUE;
}
```

Using `FirstFinder` as an example, the idea is straightforward. It keeps the evaluator and the matching result, and return a `FilterResult` of STOP when the first matching element was found.

```java
private static class FirstFinder implements NodeFilter {
    private final Element root;
    private Element match = null;
    private final Evaluator eval;

    FirstFinder(Element root, Evaluator eval) {
        this.root = root;
        this.eval = eval;
    }

    @Override
    public FilterResult head(Node node, int depth) {
        if (node instanceof Element) {
            Element el = (Element) node;
            if (eval.matches(root, el)) {
                match = el;
                return STOP;
            }
        }
        return CONTINUE;
    }

    @Override
    public FilterResult tail(Node node, int depth) {
        return CONTINUE;
    }
}
```

### The Evaluators

OK. We have gone through the traverse again via review the filtering function. Now it's time to understand how Jsoup parse a given query. We will look at `Evaluator`, `CombiningEvaluator`, and `StructuralEvaluator` first.

* `Evaluator` is a abstract class with 39 inherited subclasses which cover querying by id, name, tagName(equals or endsWith), class, attribute name, attribute value, sibling index, texts, regex match and etc. Since we have set these properties well while parsing the DOM tree, it will be very easy to implement the abstract method:
    ```java
    /**
    * Test if the element meets the evaluator's requirements.
    */
    public abstract boolean matches(Element root, Element element);
    ```

* `CombiningEvaluator` combines multiple evaluators with logic expression `And` or `Or`, which can process combinators like `",", ">", "+", "~", " "`

* `StructuralEvaluator` is used to simulate query like `div > p > span` or `div p span`. `ImmediateParent` will handle `div > p > span` while `Parent` will handle `div p span`. We will take a look at `Parent` implementation:

    ```java
    static class Parent extends StructuralEvaluator {
        public Parent(Evaluator evaluator) {
            this.evaluator = evaluator;
        }

        public boolean matches(Element root, Element element) {
            if (root == element)
                return false;

            Element parent = element.parent();
            while (true) {
                if (evaluator.matches(root, parent))
                    return true;
                if (parent == root)
                    break;
                parent = parent.parent();
            }
            return false;
        }

        @Override
        public String toString() {
            return String.format(":parent%s", evaluator);
        }
    }
    ```
    And [`QueryParser`](https://github.com/jhy/jsoup/blob/master/src/main/java/org/jsoup/select/QueryParser.java) that will parse a CSS selector into an Evaluator tree, handle above cases like this:

    ```java
    // for most combinators: change the current eval into an AND of the current eval and the new eval
    if (combinator == '>')
        currentEval = new CombiningEvaluator.And(newEval, new StructuralEvaluator.ImmediateParent(currentEval));
    else if (combinator == ' ')
        currentEval = new CombiningEvaluator.And(newEval, new StructuralEvaluator.Parent(currentEval));
    ...
    ```

### QueryParser Finally

After we explored different kinds of `Evaluator`, it will be easy to understand the parsing process in the below:

```java
private QueryParser(String query) {
    this.query = query;
    this.tq = new TokenQueue(query);
}

Evaluator parse() {
    tq.consumeWhitespace();

    if (tq.matchesAny(combinators)) { // if starts with a combinator, use root as elements
        evals.add(new StructuralEvaluator.Root());
        combinator(tq.consume());
    } else {
        findElements();
    }

    while (!tq.isEmpty()) {
        // hierarchy and extras
        boolean seenWhite = tq.consumeWhitespace();

        if (tq.matchesAny(combinators)) {
            combinator(tq.consume());
        } else if (seenWhite) {
            combinator(' ');
        } else { // E.class, E#id, E[attr] etc. AND
            findElements(); // take next el, #. etc off queue
        }
    }

    if (evals.size() == 1)
        return evals.get(0);

    return new CombiningEvaluator.And(evals);
}
```

Easier than expected, right? The implementation here is clean, elegant and easy to understand. Besides that, java doc of `Selector` is also impressive, you can play around with [Try Jsoup](https://try.jsoup.org/) with [Selector syntax](https://jsoup.org/apidocs/org/jsoup/select/Selector.html).

**-To Be Continued-**