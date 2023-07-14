# Constructor and setters

This continues the [Lombok Builder](../lombok-builder) topic and partially repeats it.

Well, this is actually the very basic of OOP but as it is not applied I have to write it.

First, OOP is not an aim but a tool. The very same result could be and is achieved with procedural style but costs more
efforts and bugs.

The only good way of instantiating an object is via its constructor **with multiple parameters**.

**Q**: Hey, what is wrong with empty constructor?

**A**: Technically speaking there is nothing. But it sounds weird from the business point of view. What is the initial
state of the object? How to distinguish two instances?

**Q**: Well, I’ll set all necessary properties via setters just after the empty constructor.

**A**: This is a wrong way. One never knows what setters should be invoked. Later the requirements might be changed, and
it is hard to check all places and add more setters. As a result the object might be at inconsistent state (some
properties won't be set).

**Q**: But it is Java Bean Specification!

**A**: Nevertheless, we shouldn't follow it. Putting it more delicately there are two ways to initialise the object.
Hibernate or Json Mapper need a technical one to create a user based on

```json
{
  "firstName": "John",
  "lastName": "Doe"
}
```

because seeing the constructor `User (String, String)` is not clear what is about. Btw, Hibernate doesn't require
setters and is smart enough to do its job via reflection but anyhow it is still a technical way.

**At the same time we should only use the business constructor(s).**

**Q**: But I have a class with 86 fields. Does it mean I have to create the constructor with 86 parameters? I would
prefer to use setters and set only necessary ones.

**A**: No. The class should have not more than 5-8 fields. If it has more than that consider to extract subclasses.
Also,
necessary ones sound weird. It doesn't mean that all fields have to be assigned but the object should be fully
initialised from the business perspective.

**Q**: Ok, somehow I’ve instantiated the object with its constructor. What is the next? Am I allowed to use setters to
change it?

**A**: No. First we need to clarify if it makes sense to change it at all. If it makes sense from the business point of
view, it is always better to use **immutable objects**. The brilliant examples of immutable design are `BigDecimal`
and `LocalDate`. Invoking their methods return **a new instance** of the class.

Btw, the core java developers also did mistakes. `java.util.Date` has a terrible design. Look at

```java
Date date = new Date(2022, 1, 1); // ha-ha, it is not January, 1st
date.setMonth(...); // WTF actually _setting_ another month means?!
```

LocalDate allows use to add or subtract some period, but it will be another LocalDate.

**Q**: Immutable objects are very interesting, but I need to manage my entities. How should I update them?

**A**: Use business mutators.

**Q**: ...and what is the difference?

**A**: At the consistency. Consider

```java
if (obj.getSomeProperty() > 5) {
    obj.setFoo("xyz");
    obj.setBar(10);
} else {
    obj.setFoo("default");
    // ops, I forgot to set the bar
}
```

where `obj` is "touched" multiple times and

```java
obj.someSelfExplainebleMethodName(5);
```

that does the very same actions but inside the class.

```java
void someSelfExplainableMethodName(Integer value) {
    if (this.someProperty > value) {
        this.foo = "xyz";
        this.bar = 10;
    } else {
        this.foo = "default";
        this.bar = 0;
    }
}
```

**Q**: But you could still miss ~~setting~~ assigning the property as well, right?

**A**: Yep. But which one is easier to test?

**Q**: So, what actually is the business mutator?

**A**: It is the method that change the object from one **consistent** state to another **consistent** state.

But don't lose you common sense, please. `file.setReadOnly(true)` or `uiComponent.setVisibility(false)` are valid
business mutators even they look pretty similar to setters (but they are not).

**Q**: This is a quite interesting topic, but I use empty constructors and setters for years and frankly saying would
like to proceed with them...

**A**: Me too. But if you’re interested in consistency this is the way how to make a step closer to it.
