---
title: "Mocking static methods (and constructors) using Mockito 3.5.0+"
date: 2020-12-30T23:22:39+01:00
draft: false
tags: ["mockito", "unit test"]
categories: ["test"]
ShowToc: true
TocOpen: true
weight: 2
---

Since version 3.4.0, Mockito allows us to [mock static methods](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#48), so you do not need to add powermock as dependency for most of your necessities.

I will not discuss why we should avoid static methods or helper class with static methods, as several articles do it quite well ([1](https://dzone.com/articles/why-static-bad-and-how-avoid), [2](https://blog.christianposta.com/testing/java-static-methods-can-be-a-code-smell/), [3](https://www.vojtechruzicka.com/avoid-utility-classes/), etc..), we need to deal with static methods in the real world of development and mockito helps us to create unit tests when these static methods are involved.

## The code

### Static helper / util class

We have a static Helper class with two methods (one with arguments and other without). 

For our example, we added a very simple code, but usually static methods have code which are harder to predict and not always junit friendly.

```java
public class GreetingsUtil {

    public static String sayFormalGoodbye() {
        return "Farewell";
    }

    public static String getFormalHello(final String name) {
        return "Hello " + name;
    }
}
```
_Full code available in the [example repo](https://github.com/tomasalmeida/articles-examples/blob/main/mockito/static/src/main/java/pro/tomasalmeida/examples/statics/util/GreetingsUtil.java) in github_


### Our class

And now, the code which uses the helper class

```java
public class PolitePerson {

    public String startConversationWith(final String name) {
        return GreetingsUtil.getFormalHello(name) + "!";
    }

    public String finishConversation() {
        return GreetingsUtil.sayFormalGoodbye() + "!";
    }
}
```
_Full code available in the [example repo](https://github.com/tomasalmeida/articles-examples/blob/main/mockito/static/src/main/java/pro/tomasalmeida/examples/statics/greetings/PolitePerson.java) in github_

## Add dependency to mockito inline

Usually for Junit5 and Mockito, we add the `mockito-core` dependency, but in this case we need to use `mockito-inline` to enable static mocking.

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-inline</artifactId>
    <version>3.6.28</version> 
</dependency>
```
_The complete version of the `pom.xml` file can be found in the [examples repo](https://github.com/tomasalmeida/articles-examples/blob/main/mockito/static/pom.xml) in github._

As you can see, I use version 3.6.28, which was the latest version available at this time. Check https://mvnrepository.com/artifact/org.mockito/mockito-inline to verify if a newer one is available when you read this article.

## Unit test

Our unit tests for need to test both methods.

### Creating a test mocking a static method with no arguments

```java
@Test
void shouldFinishConversation() {

    //given the object we want to test
    PolitePerson politePerson = new PolitePerson();

    // we use a try-with-resources block
    try (MockedStatic<GreetingsUtil> greetingsUtilMockedStatic = mockStatic(GreetingsUtil.class)) {

        //given part: pay attention that inside the when we use a method reference
        greetingsUtilMockedStatic.when(GreetingsUtil::sayFormalGoodbye).thenReturn("Bye");

        //when part: we call the part of the real code which uses the static method
        final String goodbyeMessage = politePerson.finishConversation();

        //then part: we verify our mocked method is called and the class does its part (in this case, adding the "!")
        assertEquals("Bye!", goodbyeMessage);
    }
}
```
_Full code available in the [example repo](https://github.com/tomasalmeida/articles-examples/blob/main/mockito/static/src/test/java/pro/tomasalmeida/examples/statics/greetings/PolitePersonTest.java) in github_

Some notes about the code above:
1. We statically imported the method `mockStatic` (and others).
2. The try-with-resources part is a bit verbose and forces us to create most of the code inside the `try` block.
3. The mocked method uses a method reference, in the next example, we show how to change behavior based in the argument.

### Creating a test mocking a static method with arguments

```java
@Test
void shouldStartConversation() {

    //given the object we want to test
    PolitePerson politePerson = new PolitePerson();

    // we use a try-with-resources block
    try (MockedStatic<GreetingsUtil> greetingsUtilMockedStatic = mockStatic(GreetingsUtil.class)) {

        //given part: pay attention that inside the when we verify the given argument is equals to "John"
        greetingsUtilMockedStatic.when(() -> GreetingsUtil.getFormalHello(eq("John"))).thenReturn("Hello John");
    
        //when part: we call the part of the real code which uses the static method
        final String helloMessage = politePerson.startConversationWith("John");
    
        //then part: we verify our mocked method is called and the class does its part (in this case, adding the "!")
        assertEquals("Hello John!", helloMessage);
    }
}
```
_Full code available in the [example repo](https://github.com/tomasalmeida/articles-examples/blob/main/mockito/static/src/test/java/pro/tomasalmeida/examples/statics/greetings/PolitePersonTest.java) in github_

Some notes about the code above:
1. We, again, statically imported the method `mockStatic` (and others).
2. We use a lambda expression inside the when
3. We are testing if the given argument to our static mocked test is equals to "John"

## Bonus: Mocking constructors

Some methods create new objects inside it and for tests, we could want to control how these internal objects behave. For that, in Mockito 3.5.0+ you can also mock constructors.

```java
@Test
void shouldForceAnInformalPerson() {
    // we use a try-with-resources block
    try (MockedConstruction politePersonConstructionMocked = mockConstruction(PolitePerson.class)) {

        // given part: in our example, we create the new object, but this part could be inside a method call
        PolitePerson mockedPolitePerson = new PolitePerson();

        // given part: when the mock PolitePerson finish conversation is called, an informal goodbye is sent
        when(mockedPolitePerson.finishConversation()).thenReturn("Take it easy");

        // when part: call the method
        final String informalGoodbyeMessage = mockedPolitePerson.finishConversation();

        // then part: we verify our fake message is returned and we verify the mock is called only once
        assertEquals("Take it easy", informalGoodbyeMessage);
        verify(mockedPolitePerson, times(1)).finishConversation();
    }
}
```
_Full code available in the [example repo](https://github.com/tomasalmeida/articles-examples/blob/main/mockito/static/src/test/java/pro/tomasalmeida/examples/statics/greetings/PolitePersonTest.java) in github_

Some notes about the code above:
1. Given part: we also created the object here, but in the real world, the object creation is part of the "then" part.
2. We use verify method from mockito to test the method is called only once. The `times(1)` is not needed for the value 1, but I prefer to put it explicitly to make the code more precise.

If you found any error or want to help me to improve this article, please [comment the pull request](https://github.com/tomasalmeida/tomasalmeida.github.io/pull/1)

## References:
1. [Mockito documentation static methods](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#48)
2. [Mockito documentation constructors](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#49)