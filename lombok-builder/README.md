# Lombok Builder

### TL; DR Don’t use Lombok Builder. Never

**Q**: Hey, Lombok is the awesome tool. Why are you against it?

**A**: I didn’t say I’m against Lombok. I like it too. But it has a lot of features written by angels and devils. Lombok
Builder is definitely from the dark side.

**Q**: Are you against **Builder** Pattern? It’s the well-known OOP pattern!

**A**: I’m fine with Builder Pattern. But Lombok Builder does not provide it at the right way.

**Q**: How to distinguish a good builder and a bad builder?

**A**: It’s easy. The good one should return the fully initialised object in any case. Assume we provide a `carBuilder`
for a rental agency. If a customer just wants to drive and doesn't have any special requests, `carBuilder.build()`
should provide them some car that is ready to go i.e. `car.drive()` does its job and doesn't throw any exception.
Another customer may call

```java
carBuilder
    .ofType(CarType.ECONOMY)
    .withTransmission(Transmission.AUTO)
    .withFeature(Feature.CHILD_SEAT)
    .withFeature(Feature.GPS_NAVIGATOR)
    .build();
```

and that should also be fine.

Now let’s have a deeper look at Lombok Builder. Assume we have

```java

@Builder
public class MyClass {
    private String str;
}
```

and invoke `MyClass obj = MyClass.builder().build();`
Would the `str` be initialised? No, it would be `null`!

Moreover, there is a caveat with default values. Assume we add

```java

@Builder
class MyClass {
    private String str = "xyz";
}
```

but if we call the builder again the `str` would still be `null`. Why? `@Builder.Default` was forgotten.

Also, `@Builder` conflicts with `@NoArgConstructor` and requires `@AllArgsConstructor` in this case.

But these are details. The main thing is that Lombok Builder doesn't provide **fully initialised** object and thus is
the bad builder. Don’t use it.

**Q**: So how should I initialise the objects?

**A**: There is a clear answer at OOP — use the constructor(s).

**Q**: But I have a class with 86 fields. Does it mean I have to create the constructor with 86 parameters? I would
prefer
to use the builder and only set _necessary_ ones.

**A**: No. The class should have not more than 5-8 fields. If it has more than that consider to extract subclasses.
Also, **necessary** ones sound weird. It doesn't mean that all fields have to be assigned but the object should be fully
initialised from the business perspective.

**Q**: I have the class with a few fields, but they are of the same type i.e.

```java
class Address {
    private String country;
    private String city;
    private String streetLine1;
    private String streetLine2;
}
```

so its constructor signature is `new Address(String, String, String, String)` and I’m afraid to mix them up.

**A**: This is a fair point. Modern JVM languages such as **Groovy** and **Kotlin** offer syntax sugar for that and
allow to initialise it like a map at any order i.e. `new Address(city: "Darmstadt", country: "Germany",...` but pure
Java doesn't.

IDEA also gives you a hint. Be tidy and still use the constructor.

**Q**: I follow the rule that the entities should not escape the transaction boundaries and convert them to the DTOs...

**A**: Use MapStruct for that.

**Q**: … but I don’t want to inject the dependency for a single mapping. Could I use the builder for that, i.e.

```java
Dto.builder()
    .withField(...)
    .build()
```

?

**A**: I would still suggest to use **MapStruct**. It has a killer feature that checks that all DTO’s fields are
assigned (or explicitly ignored). If you don’t want to use MapStruct anyhow, build the DTO with its constructor as well.

**Q**: I’m writing tests for the controller. It accepts some request (also a DTO). How should I set only necessary
fields at it? The client doesn't guarantee that the request is consistent. Lombok Builder seems to fit for that...

**A**: I would suggest to use the setters instead (as Spring does). But only for this particular case! — initialising
the request Dto. For domains objects the setters should be forbidden as they break the consistency.

Instead of

```java
MyRequest request = new MyRequest();
request.setFoo("user");
request.setBar(1);
```

you may write it in a bit more elegant way

```java
MyRequest request = new MyRequest()
    .setFoo("user")
    .setBar(1);
```

This is called a setter chain and turned on by `lombok.accessors.chain = true` at the `lombok.config`. There is no harm
if the setter returns `this`. But again, please use the setters only for the request DTOs.

So, there is no valid scenario for Lombok Builder. **Don’t use it. Never.**