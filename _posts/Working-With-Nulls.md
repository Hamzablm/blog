

## Dealing with nulls (Old Way)

### Assertions

Java's built-in `assert` keyword can be used to check for assumptions. It can< be a handy utility to check for nulls:

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



### java.util.Objects

### try/catch blocks

## Data outside of your control

* Document your public API
* Check for nulls only in the upper layers
* Fail fast
* Use exceptions (NullException & IllegaleException) to indicate that an invalid value had been received

## Data that you control

* Never pass null to a method (overload)
* Never return null from a method (use null object pattern or the optional type) 
* Have a good suite of tests 

## Checking for Null Using Annotations

### Spring 

### Hibernate

### Lombook



