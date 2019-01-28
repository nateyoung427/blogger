---
title: 'Behind-the-Scenes Secrets of Jsoup III: The Tree and The State Machine'
date: 2019-01-13 12:08:14
tags:
- Jsoup
categories: Code Review
keywords: Jsoup, Code Review, Github, State Machine, State Pattern
---

Ready? The challenging task finally came. We will build a tree this time so that you don't have to traverse before it was built.😅`TreeBuilder` has two implementations, `XmlTreeBuilder` is relatively easy while `HtmlTreeBuilder` is much more complex. And we will go through them today.

![](https://www.dropbox.com/s/lomj0ikzg2es1j4/build-tree.jpg?dl=1)<!-- more -->

### Overview of Package

I will skip general introduction about Compiler, Lex, Parser as there are too many articles talking about them. We will go to Jsoup source code directly. Let's have a quick look at several classes under package `org.jsoup.parser`.
* `Parser`: Facade of Jsoup parsing.
```java
    private TreeBuilder treeBuilder;  // this is the man who actually do the job
    private ParseErrorList errors;    // by default Jsoup didn't collect errors of html but you can enable this
    private ParseSettings settings;   // switches of preserveTagCase and preserveAttributeCase
```
* `Token`: Parse tokens for the Tokeniser. It's a abstract class with 6 subclasses: `Doctype`, `StartTag`, `EndTag`, `Comment`, `Character`, `EOF`

* `Tokeniser`: Read input stream into tokens. It keeps current `state` and `emitPending` as the token we are about to emit on next read, it also keeps `tagPending`, `startPending`, `endPending`, `charPending`, `doctypePending`, and `commentPending` before tokens were filled up completely

* `CharacterReader`: consumes tokens off a string, it might be inspired from `ByteBuffer` of NIO as there are similar methods like `consume()`、`unconsume()`、`mark()`、`rewindToMark()` and `consumeTo()`

* `TokeniserState` and `HtmlTreeBuilderState`: Lexing/Parsing State Machine. Jsoup applied State Pattern to implement it using enumerations, which is a classic example of design pattern usage also. Using a transition table is easy for the transition between different states but would be difficult to perform extra work during the transition

* `HtmlTreeBuilder` and `XmlTreeBuilder`: I list them in the end as they basically play a role like a manager, collaborating `Tokeniser` and Lexing, Parsing State Machine to finish the work

Lexing and Parsing State Pattern implementation would the most complex part of Jsoup. You can get to know this even just from lines of code statistics. Top 7 source files here!

|File | blank | comment | code |
|---|---|---|---|
|**org.jsoup.parser.TokeniserState.java** | 27 | 46 | 1671|
|**org.jsoup.parser.HtmlTreeBuilderState.java** | 43 | 56 | 1429|
|org.jsoup.helper.HttpConnection.java | 162 | 51 |  979|
|org.jsoup.nodes.Element.java | 156 |  604 |  727|
|org.jsoup.parser.HtmlTreeBuilder.java | 104 | 38 |  591|
|org.jsoup.select.Evaluator.java | 139 |  107 |  532|
|org.jsoup.parser.CharacterReader.java | 60 | 61 |  385|

### State Machine and State Pattern

```
⬇⬇   ⬇    ⬇  
<div>test</div>
```

![](../images/lexing-process.png)

If we use a very short code snippet as an example, each arrow would point to a state like `TagOpen`, `TagName`, `Data`, `EndTagOpen`. I just list two of them in the below:

```java
/**
 * States and transition activations for the Tokeniser.
 */
enum TokeniserState {
    Data {
        // in data state, gather characters until a character reference or tag is found
        void read(Tokeniser t, CharacterReader r) {
            switch (r.current()) {
                case '&':
                    t.advanceTransition(CharacterReferenceInData);
                    break;
                case '<':
                    t.advanceTransition(TagOpen);
                    break;
                case nullChar:
                    t.error(this); // NOT replacement character (oddly?)
                    t.emit(r.consume());
                    break;
                case eof:
                    t.emit(new Token.EOF());
                    break;
                default:
                    String data = r.consumeData();
                    t.emit(data);
                    break;
            }
        }
    },
    TagOpen {
        // from < in data
        void read(Tokeniser t, CharacterReader r) {
            switch (r.current()) {
                case '!':
                    t.advanceTransition(MarkupDeclarationOpen);
                    break;
                case '/':
                    t.advanceTransition(EndTagOpen);
                    break;
                case '?':
                    t.advanceTransition(BogusComment);
                    break;
                default:
                    if (r.matchesLetter()) {
                        t.createTagPending(true);
                        t.transition(TagName);
                    } else {
                        t.error(this);
                        t.emit('<'); // char that got us here
                        t.transition(Data);
                    }
                    break;
            }
        }
    },
    ...
}
```

Understand the idea of `Token` parse we can go to `TreeBuilder` process, we will go to the core section first, then go to `XmlTreeReader` which is easier. We will talk about `HtmlTreeReader` at last.

```java
protected void runParser() {
    while (true) {
        Token token = tokeniser.read();
        // protected abstract boolean process(Token token);
        process(token);
        token.reset();

        if (token.type == Token.TokenType.EOF)
            break;
    }
}
```

As for `Tokeniser#read()`:
```java
Token read() {
    while (!isEmitPending)
        state.read(this, reader);

    // if emit is pending, a non-character token was found: return any chars in buffer, and leave token for next read:
    if (charsBuilder.length() > 0) {
        String str = charsBuilder.toString();
        charsBuilder.delete(0, charsBuilder.length());
        charsString = null;
        return charPending.data(str);
    } else if (charsString != null) {
        Token token = charPending.data(charsString);
        charsString = null;
        return token;
    } else {
        isEmitPending = false;
        return emitPending;
    }
}
```

### XmlTreeBuilder First

After exploring the Lexing part we can go to the Parsing part now. We will go through the easier part `XmlTreeBuilder` first, focusing on its implementation of method `process(token)` in abstract class `TreeBuilder`.

`XmlTreeBuilder` basically maintains a stack and insert node according to token type:

```java
@Override
protected boolean process(Token token) {
    // start tag, end tag, doctype, comment, character, eof
    switch (token.type) {
        case StartTag:
            insert(token.asStartTag());
            break;
        case EndTag:
            popStackToClose(token.asEndTag());
            break;
        case Comment:
            insert(token.asComment());
            break;
        case Character:
            insert(token.asCharacter());
            break;
        case Doctype:
            insert(token.asDoctype());
            break;
        case EOF: // could put some normalisation here if desired
            break;
        default:
            Validate.fail("Unexpected token type: " + token.type);
    }
    return true;
}
```

As for `insert`, we will just use `StartTag` as an example in the below. The other `insert` methods accept different types of `Token` but the idea is the same.

```java
Element insert(Token.StartTag startTag) {
    Tag tag = Tag.valueOf(startTag.name(), settings);
    Element el = new Element(tag, baseUri, settings.normalizeAttributes(startTag.attributes));
    insertNode(el);
    if (startTag.isSelfClosing()) {
        if (!tag.isKnownTag()) // unknown tag, remember this is self closing for output. see above.
            tag.setSelfClosing();
    } else {
        stack.add(el);
    }
    return el;
}
```

### HtmlTreeBuilder Last

Compared to `XmlTreeBuilder`, `HtmlTreeBuilder` is much more complex. It introduces `HtmlTreeBuilderState` to process transitions. As Html is relatively loose in syntax and error tolerance, Jsoup won't simply quit when certain kinds of error detected, but choose to record them and continue. It will also try to close those tags not closed properly. Some nested tags have sequence restriction, such as a `<td>` must be under `<th>` or `<tr>`, while `tr` must be under `<tbody>` or `<table`.

There are currently 23 `HtmlTreeBuilderState` and we will use a typical Html snippet as an example, which came from Yihua Huang's work. Comments like `<!-- State: -->` will indicate what kind of `HtmlTreeBuilderState` we are at.

```html
<!-- State: Initial -->
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<!-- State: BeforeHtml -->
<html lang='zh-CN' xml:lang='zh-CN' xmlns='http://www.w3.org/1999/xhtml'>
<!-- State: BeforeHead -->
<head>
    <!-- State: InHead -->
    <script type="text/javascript">
    //<!-- State: Text -->
    function xx(){
    }
    </script>
    <noscript>
    <!-- State: InHeadNoscript -->
    Your browser does not support JavaScript!
    </noscript>
</head>
<!-- State: AfterHead -->
<body>
<!-- State: InBody -->
<textarea>
    <!-- State: Text -->
    xxx
</textarea>
<table>
    <!-- State: InTable -->
    <!-- State: InTableText -->
    xxx
    <tbody>
    <!-- State: InTableBody -->
    </tbody>
    <tr>
        <!-- State: InRow -->
        <td>
            <!-- State: InCell -->
        </td>
    </tr>    
</table>
</body>
</html>
```

As we said before, nested tags such as `tr` cannot be under `body` directly, they need to be organized under `table` or `tbody`. And DOCTYPE declaration also should not be seen in `body`. In method `boolean process(Token t, HtmlTreeBuilder tb)` of `InBody`, there is a case to handle this.

```java
case Doctype: {
    tb.error(this);
    return false;
}
```

If `head` was not closed before `body`, Jsoup will auto-close it.
```java
InHead {
    boolean process(Token t, HtmlTreeBuilder tb) {...}
    
    private boolean anythingElse(Token t, TreeBuilder tb) {
        tb.processEndTag("head");
        return tb.process(t);
    }
}
```

As the State Pattern implementation needs to handle many different kinds of cases, one can be easily lost while reading the source code. Understanding the concept and general ideas first will be of help while debugging the code to know what Jsoup is doing, but still, it's not easy and would hurt your brain😭

**-To Be Continued-**