---
title: Working With Nulls in the Java Ecosystem
author: Hamza Belmellouki
categories: [Java]
tags: [core-java, design patterns]
comments: true
---

A null value can appear anywhere in a Java application.
In this post, we'll see how to work with data you control as well as data you don't. We'll also discuss how we can use annotation libraries like JSR 380, Spring annotations, Lombok, and some patterns to handle null values.

## Dealing with nulls: Old way

### Assertions

Java's built-in `assert` keyword can be used to check for assumptions. It can be a handy utility to check for nulls:

```java
class Scratch {
    public static void main(String[] args) {
        foo(null);
    }

    public static void foo(List list) {
        assert list != null : "List is null";
        // code
    }
}
```

The above example checks if the argument of `foo` is null. If it does, then it will fail by throwing an `AssertionError` with a message: `Exception in thread "main" java.lang.AssertionError: List is null`. To enable assertion checks in your code, pass `-ea` as an option when running your code:

```shell
java -ea com.example.MyClass
```

If you don't, Java will ignore all the assertions in your code. Typically, developers enable assertion checks when they're in development and disable them when deploying to production.

### If/Else Statement
Normally we use assertions if there's a programming error. But what if it isn't? An If/Else statement can be leveraged to handle this case:
```java
if (list != null) {
    // do something with the list
} else {
    // list is null
}
```
### java.util.Objects
Introduced in Java 7 along with other static helper methods in `Objects` class. `Object.requirNonNull()` checks that the specified object reference is not null:
```java
class Scratch {
    public static void main(String[] args) {
        foo(null);
    }

    static void foo(List list) {
        Objects.requireNonNull(list, "list is null");
    }
}
```
The above example checks that the specified object is non-null and if it is, It will throw a `NullPointerException` along with the error message:
```Exception in thread "main" java.lang.NullPointerException: list is null```

Java 8 added other methods in the Objects class that can check if the object is null and vice-versa. You may prefer to use these method to increase readability and avoid typos:
```java
void foo(List list) {
    if (Objects.isNull(list)) {
        // list is null
    } else {
        // do something with the list object
    }
}
```
```java
void foo(List list) {
    if (Objects.nonNull(list)) {
        // do something with the list object
    } else {
        // list is null
    }
}
```

## Data outside of your control
We can classify data in our application into two types; Data that we can control(Internal Data) and data that is outside of our control(External Data). This section and the following one shows you the best practices you can do when dealing with these kinds of data.

Typically the presentation layer(controller) is the one responsible for getting data from the client. Therefore the data we're getting is outside of our control. In there, the controller has the responsibility for checking for null values. Follow these practices when dealing with external data:

* Document your public API
* Check for nulls only in the upper layers
* Validate method contract by adopting a fail-fast strategy
* Use exceptions (NullPointerException & IllegalArgumentException) to indicate that an invalid value had been received
* Replace the null value with some default value if it makes sense

## Data that you control
For the data that you control, remember that you shouldn't try to pass a null value to a method. For example, the method `bar` accepts two arguments. The first one is mandatory, and the second one is optional. Instead of passing a null value into the second parameter, overload the method `bar` to two methods. One that takes a single parameter, and the other takes both of the parameters(mandatory and the optional). That way, you won't need to worry about passing null values to method arguments:
```java
void bar(List list) {
        
}

void bar(List list, String msg) {
        
}
```
Another thing to remember is never to return a null value in a method. For that, you can use the **null object pattern**. For example, if your method returns a `Collection`. Instead of returning `null`, you would return an empty collection. Or you can use the `Optional` type as it provides a clear and explicit way to convey the message that there may NOT be a value without using `null`.

Finally, having a good suite of tests will definitely help you detect programming bugs caused by nulls.

## Checking for Null Using Annotations
There are third party annotations that will help you handle null values. However, in Java, there are no standard annotations to handle nulls.
Static analysis tools can leverage annotations to analyze your code at compile-time to detect a null violation error. Other annotations may throw an exception at runtime if the null condition is violated. In this section, we'll be looking at how to handle nulls using different libraries.

### Spring
Numerous null safety annotations were introduced in Spring 5.0. They are located in `org.springframework.lang` package. They can be used by static analysis tools and IntelliJ inspection tools to inspect code and generate errors and warnings at compile-time.

Spring's `@NonNull` annotation is used to check that the value passed to the parameter cannot be `null`. It can be leveraged at parameter, return value, and field level. To get null violation compile-time errors/warnings in IntelliJ, you'll need to add the annotation to IntelliJ's inspection tools. To do that go to  **Preferences | Editor | Inspections** and search for **@NonNull/@Nullable problems** and in the options section click **Configure Annotations** and then add Spring's @NotNull annotation from `org.springframework.lang` package.
Now, whenever you set `null` in `name`'s setter method. IntelliJ will generate a warning telling you that the specified setter should not accept `null` in its parameter. This example demonstrates how the use of such annotation:
```java

class Person {
    @NonNull
    private String name;
    private int age;

    public Person(@NonNull String name, int age) {
        this.name = name;
        this.age = age;
    }

    public Person() {
    }

    @NonNull
    public String getName() {
        return name;
    }

    public void setName(@NonNull String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```
The above example is a simple Java POJO class. Spring's `@NotNull` annotation is used to forbid `null` values in Person's `name` field. If you got this test method in IntelliJ, it would generate a warning telling you: `Passing 'null' argument to parameter annotated as @NotNull`:
![NotNull warning](https://i.imgur.com/jpy08KX.png)

Likewise, `@Nullable` can be used to declare that annotated elements can be null under some circumstances.

If you want to specify nullability policy as not null by default in a package level, use `@NonNullApi`.
For example, if you want to specify that all parameters and return values of `com.example.demo` package should be non-nullable by default specify that in `package-info.java`:
```java
@NonNullApi
package com.example.demo;

import org.springframework.lang.NonNullApi;
```
The above example specifies that all parameters and return values inside `com.example.demo` package are considered to be non-null by default. Note that you can override this for a specific parameter or return value by annotating them with Spring's `@Nullable` annotation.

### JSR 380
Known as Jakarta Bean Validation 2.0, is a set of constraints as annotations. Besides `@NotNull` constraint annotation, it provides others as well: `@Email`, `@NotEmpty`, `@NotBlank`, `@Positive`, `@PositiveOrZero`, `@Negative`, `@NegativeOrZero`, `@PastOrPresent` etc. You can leverage these annotations at any layer in your application. Typically you would validate your data upon creation/update of a resource in the Front layer of your application. If you're working in Spring, all you have to do is annotate your POJO class with the constraints and annotate the method parameter with `@Valid` annotation. In case if a constraint is violated, a `ConstraintViolationException` is thrown.

If your project is based on spring-boot starter validation, it will bring a dependency of Hibernate Validator, which is the reference implementation for this specification.

### Lombok
Lombok is a library used to reduce boilerplate code for data objects by generating getters, setters, constructors, hashcode, and toString, among other things.
Lombok works as an annotation post-processor. It generates code and injects it into the application. If Lombok's `@NotNull` annotation is put on a parameter, Lombok will insert a null-check at the start of the method/constructor's body, throwing a NPE with the parameter's name as a message. If put on a field, any generated method assigning a value to this field will also produce these null-checks.

In Lombok, we can configure which exception to be thrown in case of passing a `null` value. Also, the flag usage, whether compilation should generate an error or a warning.

## Null object pattern
The null object pattern can help us deal with the numerous null checks in our code. Our code may have many if statements checking for an object's nullability; thus, code becomes ugly and hard to read. You may also get various null pointer exceptions by forgetting to add a null check for your objects. The null object pattern can help by encapsulating the object's nullability logic in your object, resulting in a clean, elegant, and safe code.

The null object pattern is a particular case of the [strategy design pattern](https://hamza-jvm.me/posts/design-patterns-strategy/#2-strategy-design-pattern). Using the strategy pattern, you implement a base interface and hide the null handling logic in a unique object called the **null object**. The following diagram shows the pattern and how does it relate to the strategy pattern:
![Diagram of the pattern](https://i.imgur.com/O9NJzRy.png)

In this example, `InventoryService` is the interface that defines the contract for fetching items. Its method returns a collection of items. `ConcreteRealObject` implements that interface, which does the fetching logic maybe some processing and transformations too. `NullObject` is also an implementation that may stub `fetchItems` method by returning an empty list or some other default values.

It is recommended to leverage the null object pattern when a default value can be assigned or a default action can be taken. Please do not use it when you don't know what a default value or behavior might look like. Doing so might introduce subtle bugs in your code. So be careful and use it when appropriate.

## Optional instead of null
Typically, when developers cannot return an actual value, we tend to produce a null or throw an exception in that method. Java 8 added the `Optional` type, which is an immutable container object that may or may not contain a non-null value. An `Optional` can hold at most one value or none(null).  We can leverage its API to handle the presence and emptiness of value.

One thing to note about `Optional` is that they make clients aware that the API might return a `null` value, and this is the key difference between returning an Optional or a `null` value. It's easy to forget or ignore handling an API return value if we were returning `null`.

In this example I am populating a list of users using a static initializer:
```java
static List<User> users;
static {
    users = new ArrayList<>();
    users.add(new User("Lorem", 20));
    users.add(new User("Epsum", 22));
    users.add(new User("John", 26));
}
```

Here, I have a method that searches a user by its name. Instead of returning a null, I am returning an `Optional` so the client code that's calling the method can react to the value's presence or emptiness.
```java
static Optional<User> findUserByName(String userName) {
    final List<User> user = users.stream()
            .filter(u -> u.getName().equals(userName))
            .collect(Collectors.toList());

    if (user.size() == 1) {
        return Optional.of(user.get(0));
    } else {
        return Optional.empty();
    }
}
```

User class:
```java
class User {
   private String name;
   private int age;
    // constructor, getter, setters & toString
}
```

Client's code:
```java
public static void main(String[] args) {
    User user = findUserByName("John")
            .orElseThrow(NotFoundException::new);
    System.out.println(user);
}
```

As you can see, `orElseThrow` method returns a `User` if the user's name is "John". Else, a  `NotFoundException` exception is thrown.
There are other numerous handy Optional methods that you'll likely wanna use: `isEmpty`, `ifPresent`, map, `orElse`, etc.
The best way to learn about them is to go and explore the API at the [Javadoc](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html).

## Wrap up
In this post, we've seen how nulls can make our code unpleasant to work with. Also, we've looked over some modern best practices and patterns you can implement when working with nulls.  
If you have any feedback on this post. Please, don't hesitate to reach out to me or just say a "hello" on [Twitter](https://twitter.com/HamzaLovesJava).