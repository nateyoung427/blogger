---
title: 'To See a World in a Grain of Sand: Once Again Hello World'
date: 2019-02-11 14:42:29
tags: Exploration
categories: Exploration of Java
keywords: HelloWorld, Java, Bytecode, Java Virtual Machine, De-Compiler
---

"To see a world in a grain of sand", and we would probably see a world in the simplest "Hello World", so here we go, once again we will say Hello to the World.

![](https://www.dropbox.com/s/wt3j4nasvmqywnx/sand.jpg?dl=1)<!-- more -->

I guess all of the Java courses, tutorials start from this famous Hello World program, and this is one of those very rare programs that I can write without IDE's help:)

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

### 1. Do you know these options of javac?
After your first program was written you will execute the command below first to compile it, or you cannot run it.

```shell
javac HelloWorld.java
```

You would probably find that it's not necessary to name the file "HelloWorld.java", "Hello.java" also works. And `public class HelloWorld` also can be downgraded to `class HelloWorld`.

If you are curious enough to press `javac --help`, you will see a lot of options regarding the Java compiler, for example, we want to print Chinese edition "Hello World" and expect it apply exactly to JDK8 language level, with metadata of parameter names included in, it will look like this:

```shell
javac -encoding UTF-8 -source 8 -target 8 -parameters Hello.java
```

You've got JDK11 installed, but using the command above you are releasing class files using 1.8 features only. If you have written some stuff only available from JDK9 on you would find it cannot compile as expected.

### 2. Basics of the class file
There is a whole chapter regarding the class file format in [Java Virtual Machine specification](https://docs.oracle.com/javase/specs/jvms/se11/html/jvms-4.html), do you want to explore it a little bit?

![](https://www.dropbox.com/s/9errys0hbfqcl0e/source-byte.jpg?dl=1)

You see the bytecodes(compiled with JDK11) start with a magical, mysterious "cafe babe" and following with a value of 55 and a lot of stuff would hurt your brain. Among them, "cafe babe" is the magic, 55 points to minor version, which is mapped to JDK11. Compared to read the awesome class file format, you can also use `javap` to retrieve information for that class file:

```shell
# You would use javap -h to see how many options you have
javap -p -l -c -s -constants HelloWorld
```

You will get things like this:
```java
class HelloWorld {
  HelloWorld();                                                                                        
    descriptor: ()V                                                                                    
    Code:                                                                                              
       0: aload_0                                                                                      
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V                    
       4: return                                                                                       
    LineNumberTable:                                                                                   
      line 1: 0                                                                                        
                                                                                                       
  public static void main(java.lang.String[]);                                                         
    descriptor: ([Ljava/lang/String;)V                                                                 
    Code:                                                                                              
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;        
       3: ldc           #3                  // String Hello World                                      
       5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: return                                                                                       
    LineNumberTable:                                                                                   
      line 4: 0                                                                                        
      line 5: 8                                                                                        
}                                                                                                      
```

You can see that the instructions here are somewhat similar to the source code, with the mappings of line number of source code and instruction numbers, you might be wondering, can I restore the source from these bunches of stuff?

### 3. De-Compilers
Yes, you can. There are many decompilers, but some of them are outdated for nowadays usage, such as [JD-GUI](https://github.com/java-decompiler/jd-gui), JAD and etc, they would not work well on class file compiled with latest JDK. You may still use them, but [CFR](http://www.benf.org/other/cfr/) would be more suitable.

```shell
# java -jar cfr-0.139.jar HelloWorld.class
/*                                               
 * Decompiled with CFR 0.139.
 */                                              
import java.io.PrintStream;                      
                                                 
class HelloWorld {                               
    HelloWorld() {                               
    }                                            
                                                 
    public static void main(String[] arrstring) {
        System.out.println("Hello World");       
    }                                            
}                                                
```

You might have found that there is a slight difference with the source code and the de-compiled code(constructor method added), actually, you might be surprised to see that sometimes the generated code seems to be modified upon the source code. However, many of them are optimization from JVM and usually gains performance improvement, comparing the difference is actually interesting and would give you many insights.

### 4. How can a final variable with null value initialized again?
`System.out.println("Hello World")`, System is a class and out is one of its static attributes with the final modifier:

```java
public final static PrintStream out = null;
```

Then the problem comes, why the hack `System.out.println("Hello World")` won't throw the famous `NullPointerException`, according to the language specification, it seems that the final static variable out is impossible to be assigned to a valid value again, right?

Yes, it's right in most of the cases if you don't use the dirty reflection tricks and don't introduce the `native` buddy.

If you just want to play around, you would do like this:
```java
Field f = clazz.getDeclaredField("out");
Field modifiersField = Field.class.getDeclaredField("modifiers");
modifiersField.setAccessible(true);
modifiersField.setInt(f, f.getModifiers() & ~Modifier.FINAL);
```

However, this gonna don't work for `System`, the actual secret is hidden in these lines of code in `System.java`:

```java
private static native void registerNatives();
static {
    registerNatives();
}
```

As per the comments written above the method, "VM will invoke the initializeSystemClass method to complete the initialization for this class", go to the method of `initializeSystemClass` and you will see these lines:

```java
FileInputStream fdIn = new FileInputStream(FileDescriptor.in);
FileOutputStream fdOut = new FileOutputStream(FileDescriptor.out);
FileOutputStream fdErr = new FileOutputStream(FileDescriptor.err);
setIn0(new BufferedInputStream(fdIn));
setOut0(newPrintStream(fdOut, props.getProperty("sun.stdout.encoding")));
setErr0(newPrintStream(fdErr, props.getProperty("sun.stderr.encoding")));
```

And you will also see these 3 native methods to set `in` and `out`:

```java
private static native void setIn0(InputStream in);
private static native void setOut0(PrintStream out);
private static native void setErr0(PrintStream err);
```

So now you know that JVM does this stuff in OS level and "bypass" the `final` restriction, you would probably ask, where the hack is the OS level code that JVM will adapt with?

So here it is `System.c`[(JDK11 version)](http://hg.openjdk.java.net/jdk/jdk11/file/dde7eaaa3ddc/src/java.base/share/native/libjava/System.c).

```c
JNIEXPORT void JNICALL
Java_java_lang_System_registerNatives(JNIEnv *env, jclass cls)
{
    (*env)->RegisterNatives(env, cls,
                            methods, sizeof(methods)/sizeof(methods[0]));
}
/*
 * The following three functions implement setter methods for
 * java.lang.System.{in, out, err}. They are natively implemented
 * because they violate the semantics of the language (i.e. set final
 * variable).
 */
JNIEXPORT void JNICALL
Java_java_lang_System_setIn0(JNIEnv *env, jclass cla, jobject stream)
{
    jfieldID fid =
        (*env)->GetStaticFieldID(env,cla,"in","Ljava/io/InputStream;");
    if (fid == 0)
        return;
    (*env)->SetStaticObjectField(env,cla,fid,stream);
}
```

Here you find the back door in the comments, _"They are natively implemented because they violate the semantics of the language (i.e. set final variable)"_.

And then, you would find that it's a really a long, long road. The journey will never stop.

### The End: Stop for a while
> "To see a world in a grain of sand
> And a Heaven in a Wild Flower
> Hold Infinity in the palm of your hand
> And Eternity in an hour"

If the simplest `HelloWorld` is just a grain of sand, surely there is a world within it, maybe you have said "Hello" to it numerous times, but it doesn't mean that you have explored the world a little bit, maybe now it's the time and explore in the world, while sand would get your hands dirty, flower doesn't.

![](https://www.dropbox.com/s/arc1xfgfo4vss5z/flower.jpg?dl=1)

<script type="text/javascript">
amzn_assoc_placement = "adunit0";
amzn_assoc_search_bar = "false";
amzn_assoc_tracking_id = "javaneversleep-20";
amzn_assoc_ad_mode = "manual";
amzn_assoc_ad_type = "smart";
amzn_assoc_marketplace = "amazon";
amzn_assoc_region = "US";
amzn_assoc_title = "";
amzn_assoc_linkid = "21372c4acfda2c9cbd0a2cb43850af13";
amzn_assoc_asins = "B00K3NR6M4,1449358454,0134685997,0071350934";
</script>
<script src="//z-na.amazon-adsystem.com/widgets/onejs?MarketPlace=US"></script>
