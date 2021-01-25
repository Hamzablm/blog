---
title: Introduction to AOP and Spring AOP
author: Hamza Belmellouki
categories: [Spring]
tags: [spring-core, aop]
comments: true
---

## 1. What is AOP?

To save you the time from looking up to Wikipedia here is the definition:
> **aspect-oriented programming** (**AOP**) is a programming paradigm that aims to increase modularity by allowing the separation of cross-cutting concerns. It does so by adding additional behavior to existing code (an advice) *without* modifying the code itself, instead separately specifying which code is modified via a “pointcut” specification, such as “log all function calls when the function’s name begins with ‘set’ ”.

Don’t worry If you don’t understand the definition just read through the whole article and you’ll find an explanation for every concept in this definition.

To me, AOP is about encapsulating **system-wide concerns** or **cross-cutting concerns** (a concern that is applicable throughout the system and it affects the entire system). logging, security, and data transfer are the concerns which are needed in almost every part of a system. AOP encapsulates all of these cross-cutting concerns into something called an **Aspect**.

*Spring uses AOP internally for many things. Without proper AOP understanding, it is impossible to understand what the Spring Framework is doing under the hood.*

## 2. What Problems Does AOP Solve

AOP really comes in handy when facing one of these primary problems (and others I didn’t mention for the sake of brevity):

* Helps you encapsulate cross-cutting concerns into an aspect
* Helps you against tightly coupled dependencies in the code by extracting and moving them to an aspect
* It helps you avoid repeating code in multiple places. Imagine if we were to write a logging routine that applies to every database method call, we would have to copy and paste that block of code every single time we need it. Thus, violating the DRY principle (Don’t repeat yourself). So using Aspects removes that code duplication
* By leveraging AOP, we can maintain our application logic in a very concise format, which improves readability as well as maintainability
* The most common use cases that I’ve come into are things like transaction management, logging, caching and of course, security is a great place to use Aspects.
* etc …

## 3. AOP Concepts

These are the main concepts behind AOP. These terms are not Spring-specific:

*Aspect*: A modularization of a concern that cuts across multiple classes. It is a reusable block of code that is injected into your system at runtime. System-wide logging can be an example of such cross-cutting concerns.
*Join point*: Is a point in code in the system where the execution of an Aspect is targeted towards. These are the methods to which an advice is applied to.

*Pointcut*: A *pointcut* expression identifies the *join point* through some sort of expression matching.

Advice: This is the block of code that you execute at a *join point* that was selected by a *pointcut*. So It is your cross-cutting concern method that we’re applying to a joint point in our system.

*Target object*: The object being advised by one or more aspects.

*AOP proxy*: The object created by the AOP framework. A proxy is an intermediary object, introduced by the AOP framework, between the calling object and the target object.

## 4. Spring AOP

Spring AOP is proxy-based. Spring uses either JDK proxies (default) or CGLIB proxies to create the proxy for a given target bean. At runtime, calls to the *target object* are intercepted by the proxy, and advices that apply to the target method are executed by the proxy.

In Spring AOP, a *target object* is a bean instance registered with the spring container.

The timing of the execution of an advice depends on the type of advice. In AspectJ annotation-style, type of advice is specified by the AspectJ annotation on the advice. For instance, AspectJ’s *@Before* annotation specifies that the advice is executed before the invocation of the target method, *@After* annotation specifies that the advice is executed after the invocation of the target method, *@Around* annotation specifies that the advice is executed both *before* and *after* the execution of the target method, and so on.

## 5. Example Demo

In this simple example demo, I’ve used Spring AOP to add Logging behavior to a *CustomerAccountDAO*’s methods*.*

Here’s what *CustomerAccountDAO* looks like (Notice that its methods are annotated with *@Loggable* which we’ll use in our pointcut expression):

```java
package com.example.dao;

import com.example.aspect.Loggable;
import com.example.domain.BankAccountDetails;
import org.springframework.stereotype.Component;

@Component
public class CustomerAccountDAO {

    @Loggable
    public void createBankAccount(BankAccountDetails bankAccountDetails) {
        // database operation
    }

    @Loggable
    public void subtractFromAccount(int bankAccountId, int amount) {
        // database operation
    }
}
```

If you want to use AspectJ annotation-style for creating aspects, you need to enable support for using AspectJ annotation-style by annotating the config class with *@EnableAspectJAutoProxy. *The element also instructs the Spring AOP framework to automatically create AOP proxies for target objects.

```java
@Configuration
@EnableAspectJAutoProxy
@ComponentScan("com.example.*")
public class Config {

}
```

To add logging behavior I’ve defined an aspect which I’ve called *LoggingAspect*:

```java
package com.example.aspect;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import java.util.Arrays;

@Aspect
@Component
public class LoggingAspect {

    private static final Logger LOGGER = LoggerFactory.getLogger(LoggingAspect.class);

    @Before("@annotation(Loggable)")
    public void logMethodCall(JoinPoint joinPoint) {
        StringBuilder message = new StringBuilder("Method: ");
        message.append(joinPoint.getSignature().getName());
        Object[] args = joinPoint.getArgs();
        if (args != null && args.length > 0) {
            message.append("args=[");
            Arrays.asList(args).forEach(arg -> message.append("args=").append(arg));
        }
        message.append("]");
        LOGGER.info(message.toString());
    }
}
```

AspectJ’s *@Before* annotation specifies that the advice is executed before the invocation of the target method. It takes an argument for the pointcut expression that will identify the join points. Here the join points are any method that is annotated with @Loggable, the custom annotation I’ve defined:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Loggable {

}
```

This is the Main class where I’ve called the customerAccountDAO’s *createBankAccount* method and customerAccountDAO’s *subtractFromAccount*:

```java
package com.example;

import com.example.dao.CustomerAccountDAO;
import com.example.domain.BankAccountDetails;
import config.Config;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {

    public static void main(String[] args) {

        ApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
        CustomerAccountDAO customerAccountDAO = context.getBean(CustomerAccountDAO.class);

        customerAccountDAO.createBankAccount(new BankAccountDetails(10, 20_000));
        customerAccountDAO.subtractFromAccount(10, 1_000);

    }

}
```

If you run the application’s main method you’ll see in the console that the advice is being applied to those methods.

output:
```
INFO  com.example.aspect.LoggingAspect - Method: createBankAccountargs=[args=BankAccountDetails{accountId=10, balanceAmount=20000}]
INFO  com.example.aspect.LoggingAspect - Method: subtractFromAccountargs=[args=10args=1000]
```
### Conclusion
In this post, I attempted to clarify the concept of aspect-oriented programming and introduce you to how it relates to the Spring framework.

You can get the demo code in this [Github repository](http://bit.ly/33bpjE0). This is a Maven-based project, so it should be easy to import and run as-is.

Let us know what you think in the comments below and don’t forget to share!
