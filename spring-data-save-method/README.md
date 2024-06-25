So, the next topic I’d like to discuss is what is wrong with **Spring Data**. Well, I probably mean not the whole
project but the method `repository.save()`.

Somehow Java developers are familiar pretty well with `SQL` and `JDBC` (that is not bad, don’t get me wrong) and know
that the interaction with the DB looks like the algorithm

```
- fetch some data from the DB;
- change something based on business logic;
- save it back to the DB;
```

It is worth to mention that these steps should be wrapped by the transaction because in between of “fetch” and “save”
the concurrent call might change something that lead to the inconsistency, so the full version should be

```
- open the transaction;
- fetch some data form the DB;
- change something based on business logic;
- save it back to the DB;
- commit transaction;
```

The approach is so natural, that the developers never forget to save the results even switching to `JPA`.
But it actually offers something different, more convenient but a bit counterintuitive(?) approach. It says:

- don’t take care about IDs;
- don’t take care about the order of operations;
- don’t think of SQL;
- just focus on your business logic, `JPA` will do all technical work for you
- well, basically the only need is to notify it about brand-new objects

Technically speaking, 99.9% of our flows fit or should just fit two scenarios.

1. CREATION

```java
@Transcational
void create(Request request) {
        Entity entity = new Entity(request);
        em.persist(entity); // the brand new entity can't manage itselt, we have to nofity JPA to take care of it
}
```

As we’re Spring Data users it would probably be written as

```java
@Transactional
void create(Request request) {
        Entity entity = new Entity(request);
        entityRepository.save(entity);
}
```

2. UPDATE

```java
@Transcational
void update(ChangeRequest changes) {
       Entity entity = entityRepository.findBy(...);
       entity.apply(changes);

       // no, no, NO! Hold on! It is the managed entity already. There is no need to "save" it.
       // It is under JPA control. All its changes will be reflected to the DB automatically.
       // If not, the transaction annotation seem to be missed and it is actually even worther issue
   }
```

This is so powerful feature that allows to write something like

```java
@Transcational
void update() {
        Company company = companyRepository.findBy(...);
        company.reorganise();
}
```

It might be a terribly complex process under the hood. Departments might be created, removed and reorganised.
Employees might be reassigned accordingly. The management hierarchy might be changed. And JPA will do all technical
work for you. It will fetch (if necessary) data, builds all inserts, updates, deletes, manages their IDs and so on.
There is absolutely no need to assist it!

Now, before I write two rules regarding the `repository.save()`, let me make a pause and ask you a question.

Assume, I have a very typical parent-children `1-to-many` bidirectional relationship i.e.

```java
@Entity
class Parent {
       @Id
       Long id;

       @OneToMany(mappedBy = "parent")
       List<Child> children;
}
```
   and
   
```java
@Entity
class Child {
       @Id
       Long id;

       @ManyToOne
       Parent parent;
}
```
   
I am going to add new child to the parent, so I’m writing

```java
@Transactional
void addChild() {
       Parent parent = parentRepository.findById(...);
       
       Child child = new Child(...);
       child.setParent(parent);
       childRepository.save(child);
}
```
   
What is wrong with `save()` here? How it should look like?
