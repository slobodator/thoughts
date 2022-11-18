# Code Checklist

Even we are on Java and Spring Boot, various developers have different thoughts and preferences about approaches and
patterns.

The code is usually enclosed by NDA, hence I prepared some questions that concern me and even possible answers for them.

I'm asking absolutely seriously though they are written in a provocative manner. Have fun!

## General

- Is Lombok used?
  - definitely yes! `@Data` and `@Builder` are our best friends!
  - there is no need to use it, IDE generates all stuff
  - we use Java Records instead
  - we are on Kotlin

- Are Java Beans used?
  - indeed they are! It is a very basic thing
  - we don't use setters
  - we don't use getters
  - we are against Java Beans, so use neither setters nor getters

- Assume we're handling CRUD operations for `Users` with just `id`, `firstName` and `lastName`. Is it necessary to have
  three separate classes: `UserRequest`, `UserEntity` and `UserResponse`?
  - no, no need to create there identical classes, one class fits all
  - yes, we do that for any reason. But why?

- What Bean Mapper is used?
  - what is this and what is it needed for?
  - MapStruct
  - ModelMapper
  - Dozer
  - we prefer to have the control over the process and write the transformation code manually

## Testing

- What mock framework is used?
  - Mockito
  - PowerMock
  - WireMock
  - mocking is a bad idea

## JPA/Hibernate

- How do you manage entity's `equals()` and `hashCode()`?
  - in no way. For what?
  - in a regular way, based on all fields using `@EqualsAndHashCode`
  - based on its `id`
  - we have a `BaseEntity` for that
  - we've also read Vlad's article regarding that
  - who is Vlad?

- Is `@Embeddable` used?
  - no, haven't heard about it
  - yes, sometimes
  - it is a kind of mandatory to wrap everything with it

- What value is set for the `spring.jpa.open-in-view` property?
  - what the hell is that? what is the sense to discuss any random property?!
  - true (default)
  - false (why?)
  - well, got your concern

- How are transactions managed?
  - in no way. Let's say they are managed automatically
  - all controllers/facades are annotated with `@Transactional`
  - all services are annotated with `@Transactional`
  - there is a base class/AOP pointcut for that
  - well, it is a challenge...

- What about a SQL queries compile check?
  - do you mean the _runtime_ check?
  - Hibernate Metamodel Generator
  - QueryDSL
  - JOOQ

- How to tell that the entity property shouldn't be `null`?
  - `@Column(nullable = false)`
  - `@javax.validation.constraints.NotNull`
  - `Objects::requireNonNull`
  - I've already told you, we're on Kotlin

## REST

- What would `GET /users/7` return in case if there is no such user?
  - 404 of course
  - 204
  - 200 (what body?)
  - your option
