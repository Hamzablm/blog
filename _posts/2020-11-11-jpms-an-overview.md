---
title: JPMS&#58; An Overview
author: Hamza Belmellouki
categories: [Java]
tags: [core-java]
comments: true
---
Initially, JCP (Java Community Process) started developing JPMS(Java Platform Module System) in 2006 as JSR 277, and it was scheduled to be released in Java 7. But for some reason, it wasn't. In 2008, this JSR was dropped and replaced by JSR 376 under the umbrella of project Jigsaw. Eventually, it was released in Java 9 in 2017.

This article goes over the pain points that Java developers faced before Java 9. How can JPMS help us in solving these issues? How does JPMS impact the Java platform? And how's the tooling support in JPMS?

## Life Before Java 9
Before talking about JPMS features, let's talk about the pain points developers had before Java 9. This will help us answer an important question: Why should we develop Java applications based on the module system?

### JAR Hell
Java developers are quite familiar with the phrase "JAR Hell.", which manifests itself as misbehaving dependencies. Meaning missing or duplicated dependencies that mostly have different versions. And that could make the app crash or be subtly corrupted at runtime. 

The problem is we interpret each JAR has an identity and relationships. Whereas, before Java 9, the JVM and the compiler don't see things as we do. Instead, when the JVM loads a JAR, it considers it as a plain class-file container without any meaningful properties.

A typical example of this problem is when our app depends on two JARs, and each has a class with the same fully qualified name. This can happen for various reasons: If you accidentally depend on two JARs from the same library with different versions, this can also happen if a dependency sneaks in the form of a transitive one.
If these two different versions from the same library are present in the classpath, then all sorts of problems might happen: A runtime crash due to a missing method (For instance, the method might be renamed from version to version). We might also have an unpredictable subtle behavior due to the class's random loading since the class's behavior might have changed from version to version.

### Lack Of Encapsulation
Another major issue in Java is the lack of encapsulation. In fact, Java never had a true encapsulation since you can access any public and private members using the reflection API.

Java lacks encapsulation at the JAR level. For instance, if we declare a class as public, it will be accessible at any other JAR that depends on the JAR that owns the public class. 
But what if we want to use that class only in its JAR as an internal implementation? What if we want that class to be used only to specific JARs?

Sometimes, we notice developers depend on classes that are internal in a library that we depend on. These classes are supposed to support the public API that the library exposes. Typically these classes exist in a package suffixed by `impl` or `internal`. However, that doesn't prevent them from depending on these classes. 
You may ask, what are the consequences if we depend on these internal classes in our code? Well, this will impact the maintainability of the library's code. Thus, preventing code evolution. Or it may break our code because the library maintainer didn't care about the backward compatibility of its internal code.

## JPMS to the rescue
Now that we've talked about some of the problems that Java suffers from. Let's talk now about how JPMS can help us in solving these issues.

### Reliable Configuration
One of the primary goals of JPMS is **reliable configuration**, which enables developers to give meaningful properties to the JVM as well as the compiler. Some of these properties include a unique identifier given to our modular JAR and the dependencies' declarations with other JARs. Also, it let us specify the public API of our JAR. And much more.

These properties will be configured in what's called a **module descriptor**, which is a file named `module-info.java`. The following code snippet shows how to declare a Java module in `module-info.java`:

```
module ${module-identifier} {
   requires ${module-identifier};
   exports ${package-name};
}
```

As you can see, Java 9 has added new reserved keywords. Note that these new keywords will not impact compatibility if, for example, you have variables that are named as the reserved keyword. The point is that these keywords are strictly limited to `module-info.java` file. 

In the first line in the above module descriptor, we specify the unique module identifier for this JAR. The second line is a statement that specifies the module that your module will depend on. The third line designates the packages that your JAR exports, meaning the packages that will be considered as the public API.

### Strong Encapsulation
To address the lack of encapsulation, JPMS enforces **strong encapsulation**. It enables developers to define a true reliable public API. Note that you'll need three conditions to access a class from another module:
* Your module must depend on the module that has the desired class
* The package of the desired class must be exported
* Your class should be public

An important thing to note here is that if you're using JPMS, even when using the reflection API, your code won't be able to access private APIs if one of the above conditions isn't satisfied. That wasn't the case before Java 9, as everything was opened for reflection access.

What If you want some of your packages to be accessible only to specific modules? Well, JPMS offers **qualified exports** which lets your export specific packages to specific modules. 

Strong encapsulation helps us improve our code's maintainability because changes are straightforward if the use of implementation details packages is only internal and enforced by JPMS.

## Modular JDK
The JDK has been split into modules. Each module has exported some of its packages for public use and opened some for reflection. The following diagram highlights some of the important ones and their dependencies to the other modules:

![](https://i.imgur.com/SxgrSA3.png)

I didn't mention that there are some modules that are prefixed with `jdk`. They're not drawn in the above diagram since they're not recommended to depend on them in your code because they may vary across JDK implementers.

Note that every module you create will depend implicitly on `java.base` as it contains the core APIs that any Java program can't exist without (java.lang, java.util, etc...).

Before Java 9, you had access to all the packages that exist in these modules. Now, if you want to depend on one of these, you'll need to specify it in your module descriptor file first, and you'll only have access to exported packages and the opened ones for reflection use only.

Starting from Java 9, a tool called jlink lets you create a custom runtime JRE image just with the modules you read (depend on). If your application doesn't need modules like `java.desktop` and `java.rmi`, there's no need to include them in our custom JRE. Doing so will increase the JRE size, which is not recommended in containerized environments and smaller devices.

## Tooling Support
Tooling support is good. Build tools like Maven and Gradle do support JPMS projects. I don't know about eclipse, but IDEs like IntelliJ does understand Java 9 modular project structure. 

Adoption in frameworks and libraries is still quite slow. The Java community has to work more on it due to the value that JPMS might bring.

## Wrap Up
This post introduced you to JPMS. If you want to know more about it, I highly recommend Nicolai Parlog's book: [The Java Module System](https://www.amazon.com/Java-Module-System-Nicolai-Parlog/dp/1617294284).

If you have any feedback on this post. Please, don’t hesitate to reach out to me or just say a “hello” on [twitter](https://twitter.com/HamzaLovesJava).


## Resources
* [The Java Module System](https://www.amazon.com/Java-Module-System-Nicolai-Parlog/dp/1617294284) by Nicolai Parlog.  
* [JSR 277: JavaTM Module System](https://jcp.org/en/jsr/detail?id=277).  
* [JSR 376: JavaTM Platform Module System](https://jcp.org/en/jsr/detail?id=376).  
* [Java Modules in Practice](https://www.youtube.com/watch?v=u4VV9NSK_0Y&t=546s&ab_channel=OracleDevelopers) by Jaap Coomans.  
* [Java Modules: Why and How](https://www.youtube.com/watch?v=DItYExUOPeM&ab_channel=OracleDevelopers) by Venkat Subramaniam.  

