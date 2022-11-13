# Do not use static methods

TL;DR. Do not use static methods, please.

Q: Why not?! They are a legitimate java tool and allow to add extra functionality in a compact way.
I love them!

A: The issue is that they contradict the main idea of the OOP: an instance of the class is a model of some object
but the static method is a pure procedure that doesn't belong to any class.
They were introduced at the early ages but nowadays modern languages like Kotlin refuse of static methods.
Ok, I realize it sounds too generic, so instead of the theoretical discussion let's discuss code snippets instead.

The problem we are going to solve is quite common: assume, we have an object `foo` and somehow we want
to convert/transform/build another object `bar`. Is it fine to use

`BarUtils.toBar(Foo foo)`

... and what the alternatives are.

The very first option I'd like to suggest is to consider if it is fine (from business logic perspective of course)
to introduce just `foo.bar()`. As a side note instead of choosing the prefix (`.getBar()` or `.buildBar()`
or `.toBar()`
or something else) I would recommend to skip it at all and leave only the noun.
Assume, we have an `Integer` and want to convert it to `String`, so it would be

```java
 Integer number = 42;
 String string = number.toString();
 ```
... right? There is no need to use any static methods, isn't it?

If it is not `foo`'s responsibility to be aware of the class `Bar` and we're ready to introduce that
```java
@UtilityClass
public class BarUtils {
    public static Bar toBar(Foo foo) {
         // some implementation
     }
}
```
... consider to put the very same code at least to a regular mapper, please. Let's compare

```java
public class Service {
    public void doSmth() {
        Foo foo = ...; // get foo from somewhere
        Bar bar = BarUtils.toBar(foo);
        if (bar.getSomeValue().equals(42)) {
            // do something
        }
    }
}
```
... and
```java
@RequiredArgsConstructor
public class Service {
    private final BarMapper barMapper;

    public void doSmth() {
        Foo foo = ...; // get foo from somewhere
        Bar bar = barMapper.toBar(foo);
        if (bar.getSomeValue().equals(42)) {
           // do something
        }
    }
}
```

Q: Basically, there is no difference. It doesn't matter to use `BarUtils.toBar(foo)` or `barMapper.toBar(foo)`.
We could consider that `BarUtils` as an instance of the `BarMapper` class,  but we shouldn't create and inject it.
So, the first option is even better. 

A: That's totally wrong, unfortunately. The great advantage of the Dependency Injection is a possibility to use
another implementation if needed. How would we test the service method? I mean that branch if that `someValue` is 42?
With the static method we have to look at the `BarUtils` implementation and prepare such the `foo` value (if that's
possible!) to have the `bar` result containing that `someValue` of 42. That might be not easy!
On the contrary, with the DI it wouldn't be an issue at all. We simply mock the `barMapper` and it returns whatever
we need. 


In other words, the main concern of the static methods is that they are **hard-coded implementation** that could be
changed neither for production code nor for the tests.

The crucial point is that the implementation of the `BarMapper` must be necessarily injected
```java
public class Service {
    public void doSmth() {
        Foo foo = ...; // get foo from somewhere
        BarMapper barMapper = new BarMapper();
        Bar bar = barMapper.toBar(foo);
        if (bar.getSomeValue().equals(42)) {
            // do something
        }
    }
}
```
... is the same bad as the static one is.

This is not the end.

Could the static methods be used in some cases?

Well, as Java has a pretty limited functionality for the constructors (they could be only overloaded and that's it)
Joshua Bloch suggested to use [static factory methods](https://www.baeldung.com/java-constructors-vs-static-factory-methods#advantages-of-static-factory-methods-over-constructors) instead of them.

So, we have `Optinal.of()` and `Optional.fromNullable()` instead of `new Optional()`, `LocalDateTime.now()` and so on.

IMHO, that could be used, but we need a real good **reason** for that, and it should have a pretty limited operations.
Consider the logging. We are choosing between two evils.

The first option is a regular one.

```java
public class Foo {
   private static Logger logger = LoggerFactory.getLogger(Foo.class);
}
```

This line could and should be hidden with `@Slf4j` annotation but anyhow it has all drawbacks mentioned above.
It is a hard-coded and non-replaceable factory. We can't change its implementation, the only tweak options are provided
by the log library itself.

But the DI alternative

```java
public class Foo {
    // other properties
    private final Logger logger;

    public Foo(/* other properties */, LoggerFactory loggerFactory) {
        // other properties initialisation
        this.logger = loggerFactory.getLogger(Foo.class);
    }
}
```
... is too verbose. We would need to inject the logger factory (not the logger itself because we need to set the class
name) into the every object we want to have the logs. What to do if it is an entity, a DTO or any other class which
is created by some framework and its initiation is not under our control? Would we need to path the log factory instance
to that framework? Long story short, it could be reached but would require too many efforts just for logging, so let's
stay with the static one.

Anyhow, my point is that for our regular routine we wouldn't probably need the static factory methods.

