---
title: "Migrating From Junit4 to Junit5"
date: 2022-06-20T22:14:25+02:00
draft: false
tags: ["test", "junit5"]
categories: ["development"]
ShowToc: true
TocOpen: true
weight: 2
---

In 2019, I gave a talk at [Netcentric](https://www.netcentric.biz/) Summit about Junit5 migration and its advantages. You can check the content here:
- Video: [Netcentric | JUnit 5, mockito 2 and AEM Mocks, is it true for you?](https://www.youtube.com/watch?v=XJiZbnbu6uc)
- Blog post: [JUnit 5: leveraging new features for testing](https://www.netcentric.biz/insights/2020/07/junit5-new-features.html) 

After 3 years, I wanted to revisit the procedure to migrate from junit4 to junit5, so let's do it! The result was this [PR](https://github.com/Netcentric/accesscontroltool/pull/637).

## Steps

### 1. Dependencies

#### 1.2 Main pom.xml

In the main pom.xml, replace

```
<dependencyManagement>
    <dependencies>
...
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.xx</version>
            <scope>test</scope>
        </dependency>
...
   </dependencies>
</dependencyManagement>
```
by (check the latest version in https://mvnrepository.com/artifact/org.junit/junit-bom)
```
<dependencyManagement>
    <dependencies>
...
      <!-- Junit bill of materials -->
      <dependency>
        <groupId>org.junit</groupId>
        <artifactId>junit-bom</artifactId>        
        <version>5.8.2</version>
        <type>pom</type>
        <scope>import</scope> 
      </dependency>
...
   </dependencies>
</dependencyManagement>
```

#### 1.2 Module pom.xml


In the pom.xml of each module, replace

```
<dependencies>
...
   <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <scope>test</scope>
    </dependency>
...
</dependencies>
```
by
```
<dependencies>
...
    <dependency>
       <groupId>org.junit.jupiter</groupId>
       <artifactId>junit-jupiter-api</artifactId>
       <scope>test</scope>
    </dependency>
    <!-- temporarily needed until the tests migration -->
    <dependency>
       <groupId>org.junit.vintage</groupId>
       <artifactId>junit-vintage-engine</artifactId>
       <scope>test</scope>
    </dependency>
...
</dependencies>
```

### 2. Code Migration

I use Intellij to migrate the code.

1. Go to Refactor > Migrate packages and classes > Junit (4.x -> 5.x)
2. Try to compile your code to see if all import were correctly replaced
3. If it compiles ok, you have the basic migration done. In most of the cases we need to rewrite some tests.

### 3. Upgrading the tests

#### 3.1 @Test(expected=...)

Junit 5 has a more elegant way of testing expected exceptions. If you have

```java
@Test(expected=IllegalArgumentException.class)
public void testIfExceptionIsThrown() throws Exception {
    prepareMethod();
    methodCall();
}
```

you need to migrate to

```java
@Test
void testIfExceptionIsThrown() {
    prepareMethod();
    assertThrows(IllegalArgumentException.class, () -> methodCall());
}
```

#### 3.2 @RunWith(MockitoJUnitRunner.class)

If you have

```java
@RunWith(MockitoJUnitRunner.class)
public class MyClassTest {
```

you need to migrate to the `ExtendWith` and add a new dependency of Mockito. 

Check the latest version at https://mvnrepository.com/artifact/org.mockito/mockito-junit-jupiter.

New dependency:
```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <version>4.6.1</version>
    <scope>test</scope>
</dependency>
```

And then the new ExtendWith:
```java
@ExtendWith(MockitoExtension.class)
public class MyClassTest {
```

#### 3.3 Replace imports

Sometimes the migration fails to replace all imports. Do it manually

| Old import                                | New import                                         |
|-------------------------------------------|----------------------------------------------------|
| `import static org.mockito.Matchers.any;` | `import static org.mockito.ArgumentMatchers.any;`  | 
| `import static org.junit.Assert.`         | `import static org.junit.jupiter.api.Assertions.`  |  

#### 3.4 Unnecessary stubbings detected.

New Mockito versions are more [strict about unnecessary stubbing and they want you to use strict stubbing](http://blog.mockito.org/2018/07/new-mockito-api-lenient.html). 
If you do not want to touch your tests yet to make them more strict, you will need to add another annotation to your class or add lenient to your mock.

##### Class configuration

Add `@MockitoSettings(strictness = Strictness.LENIENT)`

```java
@ExtendWith(MockitoExtension.class)
@MockitoSettings(strictness = Strictness.LENIENT)
public class MyClassTest {
```

##### Test configuration

Add `lenient()` in front of each failed stubbing:

```
lenient().when(mock.getName()).thenReturn(name);
```

#### 3.5 Parametrized tests

In Junit5 the parametrized tests were redesigned. Take a look on [Parameterized Tests section](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests).

## References
1. [Migrating from JUnit 4 to JUnit 5 | The IntelliJ IDEA Blog](https://blog.jetbrains.com/idea/2020/08/migrating-from-junit-4-to-junit-5/)
2. [Mockito and JUnit 5 - Using ExtendWith | Baeldung](https://www.baeldung.com/mockito-junit-5-extension)
---

If you found any error or want to help me to improve this article, please [comment the pull request](https://github.com/tomasalmeida/tomasalmeida.github.io/pull/7).