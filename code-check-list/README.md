# Code Check List

**Disclaimer**. Not only you're interviewing me, I'm also curious how mature the project is, what the developers
preferences are and so on.

I realize that the code is enclosed by NDA, hence prepared some questions that concern me and even possible answers of
them.

I'm asking absolutely seriously though they are written in a provocative manner. Have fun!

## General

- Is Lombok used?
    - definitely yes!
    - there is no need to use it
    - we use Java Records instead
    - we are on Kotlin

- Are `Request`, `DomainModel` and `Response` three different classes for every business entity?
    - no, it is too verbose, one class fits all 
    - yes, sure

- What Bean Mapper is used?
    - what is this and what is it needed for?
    - MapStruct
    - ModelMapper
    - Dozer
    - we prefer to have the control over the process and write the transformation code manually

- POJO
  - we're fine with that
  - we don't use setters
  - we use neither setters nor getters

## JPA/Hibernate

- How do you manage entity's `equals()` and `hashCode()`?
    - in no way. For what?
    - in a regular way, based on all fields using `@EqualsAndHashCode`
    - based on the `id`
    - we have a `BaseEntity` for that
    - we've also read Vlad's article regarding that

- Is `@Embeddable` used?
    - no, haven't heard about it
    - yes, sometimes
    - it is mandatory to wrap everything with it

- What value is set for the `spring.jpa.open-in-view` property?
    - what the hell is that? it is not set explicitly, so the default value is used
    - yes, I know what you're asking about

- How are transactions managed?
    - in no way. Let's say, they are managed automatically
    - all controllers/facades are annotated with `@Transactional`
    - all services are annotated with `@Transactional`
    - there is a base class/AOP pointcut for that
    - well, it is a challenge...

- What about SQL queries compile check?
  - do you mean _runtime_ check?
  - hibernate metamodel generator
  - QueryDSL
  - JOOQ