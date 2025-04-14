---
title: "Unit testing part 1: What and why, different types and frameworks"
datePublished: Thu Sep 01 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j24e4011gzfs1ddh71k9m
slug: unit-testing-part-1
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617381285517/lm9oc2Sxn.jpeg
tags: csharp, unit-testing, testing, dotnet, dotnetcore

---


_This [series of posts](https://kenbonny.net/tag/better-code-with-tests/) accompanies a talk I give about unit testing with the [xUnit test framework](https://xunit.github.io/). Any reader who saw my talk can use this as a reference, others can use this as a starting point to write even better maintainable and reliable code. In my talk, I highlight patterns that work well with easily testable code and combine it all into a working application. All code from these articles can be found in a [repository](https://github.com/KenBonny/IronMan) on my GitHub account._

In this first post I talk about what tests are, why I write them, what the most common types of tests are. I finish with a comparison between the most common frameworks that are used in the .net framework.

## Define: test

A test is code outside of the production code that executes a part of said production code to verify that it behaves as intended in a predefined scenario. The key word in that sentence is _behaves_. Throughout these articles, I will talk about the behaviour of code and that is exactly what tests describe. They tell us how the code should behave in different circumstances. So if I want to know if my code does what it needs to do, then I list all the possible circumstances and the desired outcome. To determine the values to cover all the different scenarios, a technique called [boundary value analysis](http://www.guru99.com/equivalence-partitioning-boundary-value-analysis.html) can prove very valuable.

## Why do I test?

Now that I explained what a test is, let me explain the most important reasons why I write tests.

The most basic reasons is that **tests give me certainty** that my code behaves the way it's supposed to. Does that method do what I want it to, do I get exactly what I'm expecting? If something is wrong, I want to know that as soon as possible so I can fix the problem. In the end, if the tests say that everything is OK, and I'm confident I wrote good tests, then I sleep a lot better at night.

Tests make me **think more about my code**. What data do I need, what is the most efficient approach and am I doing too much in one class? Tests can easily indicate when a class is doing multiple things at once. If I have to work around a part of the code to get to the part I want to test, then that can indicate the [single responsibility principle](http://www.oodesign.com/single-responsibility-principle.html) is violated. The code I have to work around could be split up into multiple classes to separate concerns and thus make testing the desired behaviour easier. This also promotes a [loose coupling](http://www.c-sharpcorner.com/uploadfile/yusufkaratoprak/difference-between-loose-coupling-and-tight-coupling/).

**Rerunning unit tests is effortless**. Running tests manually is a lot of work. In the past, I have spent a considerable amount of time preparing a system to test a specific part of the code. If I didn't find the problem that time, I'd have to go through the whole process again. This is a time consuming, error prone process. With a unit test, I set up the environment once and I can run a specific scenario again and again.

Unit tests are **a form of documentation**. Unit tests show exactly how an application should behave. If tests are written very well, they almost read as a manual. When I'm new to a project, I always start with reading and running the tests. They explain in detail how an application works. Keep in mind that this is documentation for technical people, not end users.

Lastly, tests are a perfect **playground to experiment with code**. I can put some code in a test and run it in seconds as if it was a console application. When I want to observe the behaviour of a method or see what the impact of a specific function is, I set up a unit test that I can debug and inspect.

These benefits lead to better quality because every part of the application has been run and verified that it works as intended. If there are bugs, they surface much sooner so they can be fixed faster. The earlier I can catch bugs, the cheaper they are to fix too. As I adhere to the standards of good testing, my code will be loosely coupled, less complex, have less dependencies and is easier to maintain.

Writing good unit tests is a lot of work, but they save time later on.

**An example:**

In a project, the team and I wrote more than 1300 tests to cover a large part of the application. A part of the app would convert an XML document to an object. One day, I noticed a few properties on the return type in a converter were not being set. So I quickly mapped the fields with the corresponding properties and ran the tests. More than 50 tests failed. Upon inspection, I noticed that the new fields were optional. After adding guard clauses to verify that the corresponding field was available before mapping, all tests showed a happy green colour again. The tests prevented me from introducing bugs into the system. This saved my team, the testers, or even worse, the client from finding those bugs.

## Different types

In the introduction I promised to give an overview of the most common types of tests. Some tests check application behaviour on a high level: if this button is pushed then that field in the database is updated. Some check a much lower level of functionality: if this method is called, do I receive the correct return object.

**End to end tests** run the application as a black box. The purpose of end to end tests is to check the flow of the internal application, check that all scenarios behave as expected and all the different components integrate well. All external components need to be called; databases, file systems and the network need to be accessible. The code should behave as it is intended from start to finish.

**Integration tests** connect to external systems such as databases or web services and see if the system under development can interact with other systems. They test whether stored procedures are called correctly, web services implement the correct contract and file system access rights are correct.

**Component tests**, also called **functional tests**, run complete components to see that they work as expected. All dependant interfaces and classes use actual implementations. Say the code contains a `ValidationChecker` that verifies input by executing a number of `ValidationRules`. A component test would select specific input and run it through the `ValidationChecker` with all the rules loaded to see whether the input is valid.

**Unit tests** check whether individual methods and functions behave as expected. All dependant interfaces and classes are mocked so the test focuses solely on one class. Where a component test loads all validation rules, unit tests check each validation rule separately and test the `ValidationChecker` with mocked rules.

Lastly, **regression tests** are all previous tests that are run after a code change. They ensure that the application still behaves as intended after something has been updated or changed.

## Most popular frameworks

There are three main frameworks I've worked with over the past few years: MSTest, [NUnit](http://www.nunit.org/) and [xUnit](https://xunit.github.io/).

The first one is MSTest, which actually is not supported anymore (for several years). Internally, Microsoft has even moved to xUnit for a large number of tests in their .NET framework.

```
[TestClass]
public class MsTest
{
  private string _first;
  private string _second;
 
  [TestInitialize]
  public void Init()
  {
    _first = &quot;The beginning comes before&quot;;
    _second = &quot; the end.&quot;;
  }
 
  [TestMethod]
  public void TestMethod()
  {
    // arrange
    var builder = new StringBuilder(_first);
    // act
    builder.Append(_second);
    // assert
    Assert.AreEqual(&quot;{_first}{_second}&quot;, builder.ToString());
  }
}
```

Another very popular test framework is NUnit. One of the creators of xUnit also created NUnit. It's similar to MSTest, but it has more features and it is a lot more optimized. NUnit is very active as I write this, so this is an excellent unit test framework as well.

```
[TestFixture]
public class NUnitTests
{
  private string _first;
  private string _second;
 
  [Setup]
  public void Init()
  {
    _first = &quot;The beginning comes before&quot;;
    _second = &quot; the end.&quot;;
  } 
 
  [Test]
  public void TestMethod()
  {
    // arrange
    var builder = new StringBuilder(_first);
    // act
    builder.Append(_second);
    // assert
    Assert.AreEqual($&quot;{_first}{_second}&quot;, builder.ToString());
  }
}
```

xUnit's inventors, [Brad Wilson](http://bradwilson.typepad.com/) and [James Newkirk](http://jamesnewkirk.typepad.com/), figured they could do better and with a lot more object oriented principles in mind. In my opinion, they succeeded phenomenally.

```
public class XUnitTests
{
  private readonly string _first;
  private readonly string _second;
 
  public XUnitTests()
  {
    _first = &quot;The beginning comes before&quot;;
    _second = &quot; the end.&quot;;
  }
 
  [Fact]
  private void TestMethod()
  {
    // arrange
    var builder = new StringBuilder(_first);
    // act
    builder.Append(_second);
    // assert
    Assert.Equal($&quot;{_first}{_second}&quot;, builder.ToString());
  }
}
```

The first notable difference is that there's no attribute on the class. This is one of the philosophies that are at the core of xUnit. Write as little as possible. Besides that, the initialization of the fields happens in the constructor. Each test (which is called a fact) gets its own instance of the test class so the constructor gets called before each test is run. The tests are also run in parallel to speed up test execution. As the fields are now instantiated in the constructor, they can be made readonly, which is again an improvement. The other test frameworks also require all tests comply with the signature of `public void ...`. xUnit also accepts non `public void` methods. This means that I can have test methods next to actual code as `private void...`, but this is an anti-pattern and should never be done. The reason being that it would bloat projects with unnecessary dependencies on the xUnit framework. Keep code and tests separate.

Visit the [xUnit comparison page](https://xunit.github.io/docs/comparisons.html) for more information about the differences between MSTest, NUnit and xUnit.

In the next part, I will describe starting with Test Driven Design. If anybody notices any inconsistenties, please contact me on [twitter](https://twitter.com/bonny_ken/) or via [email.](mailto:bonny.ken@gmail.com)
