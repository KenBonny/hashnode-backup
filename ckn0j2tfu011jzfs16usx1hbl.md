---
title: "Unit testing part 5: Even less code with initialization and cleanup"
datePublished: Mon Sep 26 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j2tfu011jzfs16usx1hbl
slug: unit-testing-part-5
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1618405142877/ulkrBA42U.png
tags: csharp, unit-testing, testing, dotnet, dotnetcore

---


_This [series of posts](https://kenbonny.net/tag/better-code-with-tests/) accompanies a talk I give about unit testing with the [xUnit test framework](https://xunit.github.io/). Any reader who saw my talk can use this as a reference, others can use this as a starting point to write even better maintainable and reliable code. In my talk, I highlight patterns that work well with easily testable code and combine it all into a working application. All code from these articles can be found in a [repository](https://github.com/KenBonny/IronMan) on my GitHub account._

After optimizing tests with data, it's time to streamline unit tests with initialization and cleanup. xUnit can even reuse code across tests. This can be convenient, but is very dangerous.

## The setup

Before diving into the tests, I have added a class `MissileFactoryFixture`. That class's purpose will become clear throughout this chapter.

## Initialization

```
[Fact]
public void When_launching_missile_then_decrease_missile_count_by_1()
{
  // arrange
  var mockMissile = new Mock<IMissile>();
  var launcher = new Launcher(new List<IMissile> {mockMissile.Object});
  // act
  launcher.Launch();
  // assert
  Assert.Equal(0, launcher.MissileCount);
  Assert.True(launcher.MissileCount == 0, "Missile was not launched");
}
```

The initialization of the `Mock<IMissile>` can be moved to the initialization part of the test, this is code that I could reuse. Since each test gets its own instance of the test class, the constructor will be called each time. This is where initialization takes place. The first change I make is to add a `private readonly field Mock<IMissile> mockMissile`. In the constructor I instantiate this field. I remove the local variables and replace them with a reference to the field I just created. I run the tests again to see that everything still works.

How do I know that I get a new instance of mockMissile with each test? By adding a breakpoint and debugging the tests. I see that before each test, I hit the breakpoint in the constructor which overrides the value in the field.

After my changes, the test class looks like:

```
private readonly Mock<IMissile> mockMissile;
public LauncherTests()
{
  mockMissile = new Mock<IMissile>;
}
// tests go here
```

## Test Fixtures

What about more resource intensive setup that could be reused across different tests such as a database mock. xUnit supports creating fixtures that will be persisted across test runs. This should be reserved for long running setup that can be reused and should not be changed during test execution. If one test alters the state of the object, then all subsequent tests will be impacted and cannot be trusted to be correct. Be very careful when using fixtures. A best practice is to make the test fixture [immutable](https://en.wikipedia.org/wiki/Immutable_object).

This is where the `MissileFactoryFixture` class is for. I can implement this class any way I see fit, but the most convenient way is to have properties that contain the objects I need and then populate them in the constructor of the `MissileFactoryFixture` class. This ensures that the code runs when xUnit creates an instance of `MissileFactoryFixture`.

In my example, I move the instantiation of the `MissileFactory` to the fixture class. The fixture is then injected into the test class through the constructor. By implementing the `IClassFixture<MissileFactoryFixture>` interface on the test class, xUnit is notified that the `MissileFactoryFixture` needs to be injected. `IClassFixture` is an empty interface just to indicate to xUnit which classes need to be injected. I save the injected fixture in a field so it is available within my tests and use the `fixture.MissileFactory` across multiple tests.

Multiple fixtures can be injected by adding multiple `IClassFixture<>` interfaces to a test class. Such a scenario should not happen a lot. If it does, maybe it is more worthwhile to rethink the architecture of the application.

## Cleanup

To clean up after a test, xUnit expects an implementation of the `IDisposable` interface. The `Dispose` method will be called after each run. This is the place to clean up after a test. I have used this in the past to remove copied files or roll back database transactions so no changes to databases would be persisted.

The class now looks like this:

```
private readonly MissileFactoryFixture fixture;
public MissileFactoryTests(MissileFactoryFixture fixture)
{
  this.fixture = fixture;
}
public void Dispose()
{
  this.fixture = null;
}
// tests go here
```

There is one more part to this and that is when I would want to [use the same fixture across different test classes](https://xunit.github.io/docs/shared-context.html#collection-fixture). Implementing this is not difficult but is not used much because it is very dangerous as there are now even more places where the shared content can be altered. Thus I shall be skipping this subject. Know that it is available should it ever become convenient to use.

## Conclusion

Test initialization can be done in the constructor, the cleanup in the `Dispose` method of the `IDisposable` interface. Reusing test setup can be accomplished by test fixtures that are injected into the test with the help of the `IClassFixture<>` interface. It is a more dangerous way to implement the tests because of possible side effects. I recommend making test fixtures immutable.

This is all nice and well, but what about older code that doesn't use all these fancy tricks and separation? In the upcoming blog post, I'll address the issue of older code. If anybody notices any inconsistenties, please contact me on [twitter](https://twitter.com/bonny_ken/) or via [email](mailto:bonny.ken@gmail.com).
