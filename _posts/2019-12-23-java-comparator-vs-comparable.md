---
title: Core Java&#58; Comparator vs. Comparable
author: Hamza Belmellouki
categories: [Java]
tags: [core-java]
comments: true
--- 


When working with custom objects, we often want to compare them based on some pre-defined criteria. That’s why we use the `Comparable` interface to implement a *natural ordering* for our custom objects.

Also, we often need to specify a *total ordering* if there is no *natural ordering* implemented by our custom object or override the *natural ordering* defined by the object, so we use a `Comparator` interface to do such a thing.

## 2. The Example Demo

In the example demo, we’ll show how to compare `Person` objects using *natural ordering* implemented by the `Comparable` interface and also to override that *natural ordering* by using a new ordering implemented by the `Comparator` interface.

Assume you have a `Person` class as demonstrated below:

```java
class Person {
    private int age;
    private String firstName;
    private String lastName;

    public Person(int age, String firstName, String lastName) {
        this.age = age;
        this.firstName = firstName;
        this.lastName = lastName;
    }
    
    // getters and setters
    
    @Override
    public String toString() {
        return "Person{" +
                "age=" + age +
                ", firstName='" + firstName + '\'' +
                ", lastName='" + lastName + '\'' +
                '}';
    }
}
```

## 3. Comparable Interface

### 3.1 Definition

You define the *natural ordering* for your objects by implementing the Comparable’s `compareTo(T o)` method in your custom objects. For example, if we want to compare the `Person` object based on the *first name* we’ll implement the `Comparable` interface like so:

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

class Person implements Comparable<Person> {
    private int age;
    private String firstName;
    private String lastName;

    public Person(int age, String firstName, String lastName) {
        this.age = age;
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return "Person{" +
                "age=" + age +
                ", firstName='" + firstName + '\'' +
                ", lastName='" + lastName + '\'' +
                '}';
    }

    @Override
    public int compareTo(Person person) {
        return this.firstName.compareTo(person.firstName);
    }

    public static void main(String[] args) {

        // people to sort
        List<Person> people = new ArrayList<>(List.of(
                new Person(20, "Hamza", "Belmellouki"),
                new Person(60, "Allan", "Truck"),
                new Person(40, "Zidan", "Kemero"),
                new Person(30, "Cindy", "Mahbd")
        ));
        people.forEach(System.out::println); //unsorted

        System.out.println("----------");

        Collections.sort(people);// sorted
        people.forEach(System.out::println);

    }
}
```

If you run the program the output will be:
```
Person{age=20, firstName=’Hamza’, lastName=’Belmellouki’}
Person{age=60, firstName=’Allan’, lastName=’Truck’}
Person{age=40, firstName=’Zidan’, lastName=’Kemero’}
Person{age=30, firstName=’Cindy’, lastName=’Mahbd’}
 — — — — — 
Person{age=60, firstName=’Allan’, lastName=’Truck’}
Person{age=30, firstName=’Cindy’, lastName=’Mahbd’}
Person{age=20, firstName=’Hamza’, lastName=’Belmellouki’}
Person{age=40, firstName=’Zidan’, lastName=’Kemero’}
```

As you can see the `compareTo` method implements the sorting strategy based on the lexicographic order. It calls the `compareTo` implementation that is defined in the `String` class which compares two strings lexicographically.

Our `compareTo` implementation returns the value 0 if the `person.firstName` is equal to `this.fisrtName`, a value less than 0 if `this.firstName` is lexicographically less than the `person.firstName`; and a value greater than 0 if `this.fisrtName` is lexicographically greater than the string argument.

Notice `Collections.sort` method accepts a list of Person, it sorts the list of Person based on the *natural ordering*.

If the Person class does not implement `Comparable` then a compile-time error occurs because the `Collections.sort` method defines its generic type parameter to extend the `Comparable` interface (you know generic type safety is here to rescue us from `ClassCastException`! That’s another subject, maybe for another blog).

### 3.2 Best practices when using Comparable:

These are some best practices when using a `Comparable`:

Sometimes you may see `compareTo` method that relies on the fact that the difference between two values is negative if the first is less than the second, zero if the values are equal, and positive if the first value is greater. For example, this code violates `compareTo` method contract(transitivity):
```java
@Override
public int compareTo(Person p) {
    return this.age - p.age;
}
```

Don’t use this technique. It is filled with danger from integer overflow and [IEEE 754](https://en.wikipedia.org/wiki/IEEE_754) floating arithmetic artifacts.

Instead, Always either use the static Integer, Double, Float’s `compare` method. For example:
```java
@Override
public int compareTo(Person p) {
    return Integer.compare(this.age, p.age);
}
```

## 4. Comparator Interface

### 4.1 Definition

If we want to define a *total ordering* on an object that has no *natural ordering* or if we want to override the natural *ordering* then we leverage the `Comparator` interface to do so. For example, if we want to override the natural ordering of `Person` objects and compare the `Person` object based on the age we’ll implement the `Comparator` interface like so:

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

class Person implements Comparable<Person> {
    private int age;
    private String firstName;
    private String lastName;

    public Person(int age, String firstName, String lastName) {
        this.age = age;
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return "Person{" +
                "age=" + age +
                ", firstName='" + firstName + '\'' +
                ", lastName='" + lastName + '\'' +
                '}';
    }

    @Override
    public int compareTo(Person person) {
        return this.firstName.compareTo(person.firstName);
    }

    public static void main(String[] args) {

        // people to sort
        List<Person> people = new ArrayList<>(List.of(
                new Person(20, "Hamza", "Belmellouki"),
                new Person(60, "Allan", "Truck"),
                new Person(40, "Zidan", "Kemero"),
                new Person(30, "Cindy", "Mahbd")
        ));
        people.forEach(System.out::println); //unsorted

        System.out.println("----------");

        Comparator<Person> cmp = (o1, o2) -> Integer.compare(o1.age, o2.age);

        people.sort(cmp); // sorted

        people.forEach(System.out::println);
    }
}
```

If you run this code the output will be:

    Person{age=20, firstName=’Hamza’, lastName=’Belmellouki’}
    Person{age=60, firstName=’Allan’, lastName=’Truck’}
    Person{age=40, firstName=’Zidan’, lastName=’Kemero’}
    Person{age=30, firstName=’Cindy’, lastName=’Mahbd’}
     — — — — — 
    Person{age=20, firstName=’Hamza’, lastName=’Belmellouki’}
    Person{age=30, firstName=’Cindy’, lastName=’Mahbd’}
    Person{age=40, firstName=’Zidan’, lastName=’Kemero’}
    Person{age=60, firstName=’Allan’, lastName=’Truck’}

Note that the list is sorted by age. In line 43 you can see we have implemented the single abstract `compare` method on the `Comparator` interface using a lambda. Our implementation of the [Comparator#compare](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Comparator.html#compare(T,T)) method takes two person objects and returns a negative value if `o1.age` is less than the `o2.age`; 0 if they are equal; positive integer in the `o1.age` is greater than the `o2.age`.

In line 45 I’ve called [List#sort](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html#sort(java.util.Comparator)) method which sorts the list based on the `Comparator` implementation that we’ve passed to it.

### 4.2 Best practices when using Comparator:

These are some best practices when using a `Comparator`:

The practice from `Comparable` interface applies here: Don’t use the difference to compare objects, instead always rely on the static Integer, Double, Float’s `compare` method to compare numeric values:
```java
Comparator<Person> comparator = (p1, p2) -> Integer.compare(p1.getAge(), p2.getAge());
```

Failing to do so could result in an integer overflow and [IEEE 754](https://en.wikipedia.org/wiki/IEEE_754) floating arithmetic artifacts.

The second best practice that is recommended is to avoid chaining if-else statements that result from additional comparison if objects were equal on the previous comparison(s). To illustrate, in this code snippet we implemented a `Comparator` that compares the person objects based first on their first name if they were equal then we move to the last name then to age:

```java
public static void main(String[] args) {

        // people to sort
        List<Person> people = new ArrayList<>(List.of(
                new Person(20, "Hamza", "Belmellouki"),
                new Person(50, "Hamza", "Belmellouki"),
                new Person(44, "Hamza", "Belmellouki"),
                new Person(60, "Allan", "Truck"),
                new Person(40, "Zidan", "Kemero"),
                new Person(30, "Cindy", "Mahbd")
        ));
        people.forEach(System.out::println); //unsorted

        System.out.println("----------");

        // Don't do this!
        Comparator<Person> cmp = (p1, p2) -> {
          int result = p1.firstName.compareTo(p2.firstName);
            if (result == 0) {
                result = p1.lastName.compareTo(p2.lastName);
                if (result == 0) {
                    return Integer.compare(p1.age, p2.age);
                } else {
                    return result;
                }
            } else {
                return result;
            }
        };

        people.sort(cmp); // sorted

        people.forEach(System.out::println);

    }
}
```

Output:

    Person{age=20, firstName='Hamza', lastName='Belmellouki'}
    Person{age=50, firstName='Hamza', lastName='Belmellouki'}
    Person{age=44, firstName='Hamza', lastName='Belmellouki'}
    Person{age=60, firstName='Allan', lastName='Truck'}
    Person{age=40, firstName='Zidan', lastName='Kemero'}
    Person{age=30, firstName='Cindy', lastName='Mahbd'}
    ----------
    Person{age=60, firstName='Allan', lastName='Truck'}
    Person{age=30, firstName='Cindy', lastName='Mahbd'}
    Person{age=20, firstName='Hamza', lastName='Belmellouki'}
    Person{age=44, firstName='Hamza', lastName='Belmellouki'}
    Person{age=50, firstName='Hamza', lastName='Belmellouki'}
    Person{age=40, firstName='Zidan', lastName='Kemero'}

You don’t have to use this old pattern when you want to do such a comparison because it is hard to read, error-prone code and lacks robustness. Instead, you would use Java 8 Comparator’s `comparing` and `thenComparingXXX` methods:

```java
public static void main(String[] args) {

        // people to sort
        List<Person> people = new ArrayList<>(List.of(
                new Person(20, "Hamza", "Belmellouki"),
                new Person(50, "Hamza", "Belmellouki"),
                new Person(44, "Hamza", "Belmellouki"),
                new Person(60, "Allan", "Truck"),
                new Person(40, "Zidan", "Kemero"),
                new Person(30, "Cindy", "Mahbd")
        ));
        people.forEach(System.out::println); //unsorted

        System.out.println("----------");

        Comparator<Person> cmp = Comparator.comparing(Person::getFirstName)
                .thenComparing(Person::getLastName)
                .thenComparingInt(Person::getAge);

        people.sort(cmp); // sorted

        people.forEach(System.out::println);

}
```

This code does that same thing but it is much more readable and less error-prone. Here on line 16, the static Comparator’s `comparing` method takes a key extractor which is a `Function` object implementation used to extract the Comparable sort key.

Notice on line 18 I used `thenComparingInt` instead of `thenComparing` because the former takes a `ToIntFunction` object, thus there will be no boxing involved because its abstract method returns an int primitive, not a boxed int.

## 5. Comparable vs. Comparator

You'll want to use the Comparable interface when we want a default order. When using a Comparable interface we don’t need to make any code changes at the client-side. For example, `Collections#sort` method automatically uses the `compareTo()` method of the class. For Comparator, the client needs to provide the Comparator class to use in `compare()` method.

Both of these interfaces are part of the Java Collection framework.

There are several reasons to use a Comparator even if we already have a Comparable implemented:

* Specify different comparison strategies which aren’t possible when using *Comparable*

* To avoid changing/adding code to our custom classes

* Often, it is impossible to modify the source code of the class whose objects we want to sort, thus it is impossible to use Comparable in such circumstances.

## 7. Conclusion

In this article, we saw how to use Comparable to implement the natural ordering and how to override it using a Comparator. We saw well-known best practices to use when implementing them. In the next article, there will be more Core Java. Stay tuned!

Let us know what you think in the comments below and don’t forget to share!
