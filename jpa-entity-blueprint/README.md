# JPA Entity Blueprint

As I think that the incorrect **entities** are the cause of many issues, and it is extremely difficult to refactor them
later (as they affect the entire application), here is a blueprint that might be used as a better start.

To read more about `equals()` and `hashCode()` implementations, please, have a look at an
awesome [Vlad Mihalcea post](https://vladmihalcea.com/the-best-way-to-implement-equals-hashcode-and-tostring-with-jpa-and-hibernate/).

```java
// @Data -- is not applicable at all, it may cause a lot of issues
// @Setter -- please, don't use them
// @EqualsAndHashCode -- the entity requires its special handling, see below
// @AllArgsConstructor -- doesn't fit because it includes ID
@Getter
// Getters are allowed, but please wrap nullable fields with optional. Also, the collections require explicit operations
@ToString(onlyExplicitlyIncluded = true)
// this is mainly for debug but take care of the collections with a lot of entries
@NoArgsConstructor(access = AccessLevel.PROTECTED) // this one is for Hibernate
@Entity
//@Table is optional actually, by default it is the class name
public class Foo { // It is a matter of taste, but I see not reason to add Entity suffix to the class name
    @Id
    @SequenceGenerator(name = "fooSeq", sequenceName = "FOO_SEQ")
    @GeneratedValue(generator = "fooSeq", strategy = GenerationType.SEQUENCE)
    // please, don't skip the strategy. It causes the issue with H2 dialect
    @ToString.Include
    private Long id;

    @ToString.Include
    private long payload; // assuming the payload is mandatory there is no need to use Long for it

    @ToString.Include
    private String description; // assuming it is optional and might be null

    // as we explicitly include the fields for toString() it will be omitted
    @OneToMany(mappedBy = "foo", cascade = CascadeType.ALL, orphanRemoval = true)
    @OrderBy("someBarField")
    private List<Bar> bars = new ArrayList<>();

    // Here are some public constructors for business purpose
    // Please note that ID and the collection are excluded
    public Foo(long payload, String description) {
        this.payload = payload;
        this.description = description;
    }

    public Foo(long payload) {
        this(payload, null);
    }

    // how to initialize the collection?
    public Foo(long payload, String description, Collection<Bar> bars) {
        this(payload, description);

        // we should not substitute the collection but to modify it to let Hibernate to track the relations
        bars.stream().forEach(b -> this.bars::add);
    }

    // it gives a clear signal that the description is optional
    public Optional<String> getDescription() {
        return Optional.ofNullable(description);
    }

    // wrap the collection to prevent its modification
    public List<Bar> getBars() {
        return Collections.unmodifiableList(bars);
    }

    // methods for the collection modification
    public addBar(Bar bar) {
        this.bars.add(bar);
        bar.setFoo(this);
    }

    public removeBar(Bar bar) {
        this.bars.remove(bar);
        bar.setFoo(null);
    }

    // ID based equals
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Foo foo)) return false;
        return id != null && Objects.equals(id, foo.getId()); // it is crucial not to use foo.id here
    }

    // it couldn't rely on ID because it is null for newly created entities, so it uses the constant
    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}
```

The very same but without comments

```java

@Getter
@ToString(onlyExplicitlyIncluded = true)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
public class Foo {
    @Id
    @SequenceGenerator(name = "fooSeq", sequenceName = "FOO_SEQ")
    @GeneratedValue(generator = "fooSeq", strategy = GenerationType.SEQUENCE)
    @ToString.Include
    private Long id;

    @ToString.Include
    private long payload;

    @ToString.Include
    private String description;

    @OneToMany(mappedBy = "foo", cascade = CascadeType.ALL, orphanRemoval = true)
    @OrderBy("someBarField")
    private List<Bar> bars = new ArrayList<>();

    public Foo(long payload, String description) {
        this.payload = payload;
        this.description = description;
    }

    public Foo(long payload) {
        this(payload, null);
    }

    public Foo(long payload, String description, Collection<Bar> bars) {
        this(payload, description);
        bars.stream().forEach(b -> this.bars::add);
    }

    public Optional<String> getDescription() {
        return Optional.ofNullable(description);
    }

    public Set<Bar> getBars() {
        return Collections.unmodifiableSet(bars);
    }

    public addBar(Bar bar) {
        this.bars.add(bar);
        bar.setFoo(this);
    }

    public removeBar(Bar bar) {
        this.bars.remove(bar);
        bar.setFoo(null);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Foo)) return false;
        Foo foo = (Foo) o;
        return id != null && Objects.equals(id, foo.getId());
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}
```

And here is a blueprint for the embeddable objects. The rules regarding its details are the same.

```java

@Embeddable
@Getter
@ToString(onlyExplicitlyIncluded = true)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor // this time it is a good idea to use it as there is no generated ID
// @Entity -- it is not an entity with ID
// @EqualsAndHashCode this time there is no blueprint for them
// if the object is immutable it might build these methods on all these fields
// otherwise it is a generic question what the equality actually means
public class Emb {
}
```

without the comments

```java

@Embeddable
@Getter
@ToString(onlyExplicitlyIncluded = true)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
public class Emb {
}
```
