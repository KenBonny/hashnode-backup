---
title: "Unit testing part 6: The horrors of older code"
datePublished: Tue Oct 04 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j2wv3011mzfs19hbrd4o0
slug: unit-testing-part-6
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1618404960328/WHa2h9NVq.png
tags: csharp, unit-testing, testing, dotnet, dotnetcore

---


_This [series of posts](https://kenbonny.net/tag/better-code-with-tests/) accompanies a talk I give about unit testing with the [xUnit test framework](https://xunit.github.io/). Any reader who saw my talk can use this as a reference, others can use this as a starting point to write even better maintainable and reliable code. In my talk, I highlight patterns that work well with easily testable code and combine it all into a working application. All code from these articles can be found in a [repository](https://github.com/KenBonny/IronMan) on my GitHub account._

Every line of code I write from now on will have a dozen tests checking every possible behaviour. The problem is my legacy work project, it's not written with tests in mind. Such a project is not easy to test. Great! Nothing is as fun as a challenge (even if I procrastinate a bit before tackling it).

## Testing older code

The main problem with such code is that it's not always possible to test and that's sometimes very hard to determine up front. A lot of class instantiations can mean that it's difficult to work around or it's going to be very time consuming. Sometimes I'm afraid to touch such code out of fear that something else will break when I make a change. These situations just scream for unit tests so I can easily see the implications in other parts of the code. Undesired behaviour will break tests, making me aware of bugs very early.

That said, adding tests will most of the time require changing the old code. I try to keep such changes to a minimum, but they are inevitable. Over time I find it easier to split code up and thus make it more testable. This is again a time consuming process and should be done gradually as to not change too much at once.

## The old launcher

As luck will have it, there is a launcher implementation according to some simple, but bad practices. When instantiating the `OldLauncher`, it creates a list of `OldMissiles` to fire. Lets see how I can add tests to this `OldLauncher`.

```
public class OldLauncher
{
  private List<OldMissile> missiles;
  public int MissileCount => missiles.Count;
  public OldLauncher()
  {
    this.missiles = new List<OldMissile> {new OldMissile(), new OldMissile()};
  }
  public void Launch()
  {
    var missile = missiles.First();
    missiles.Remove(missile);
    if (!missile.Fire())
    {
      throw new Exception("Missile could not be launched.");
    }
  }
}
```

First I add a test that calls the `Launch` method and checks that the `MissileCount` decreases by one to check that the missile that gets launched is actually removed as well. This code looks a lot like the original test I wrote.

## Updating dependant code

A problem arises when I want to check when the `Fire` method should return false (the `OldMissile` only returns true). This can only be done by mocking the missile. To do that, I'm going to need to change the `OldLauncher` **without impacting existing code**. One way is by adding an additional constructor that injects a list of missiles into the property.

```
public OldLauncher(List<OldMissile> missiles)
{
  this.missiles = missiles;
}
```

By adding such a constructor, I can inject mock objects. Using the Moq framework or inheritance to create a fake `OldMissile`, I can add code that lets me specify that the `Fire` method returns false. There will be situations where I will need to update those dependent classes to mark certain methods as `virtual` so I can more easily override them. This would make for a great opportunity to hide such concerns behind an interface. I'll have to decide case by case to see what works best with the rest of the code.

In this case, I'll make the `Fire` method of the `OldMissile` virtual so I can verify that it is being called by the `OldLauncher` through the Moq framework. Now I can let the `Fire` method return false and test getting an exception. With xUnit, there is an `Assert` method that expects an exception will be thrown. The method will return the exception so it can be verified. It also allows to verify other prerequisites, for example wether the `Fire` method was called.

```
[Fact]
public void When_fire_fails_then_expect_exception()
{
  // arrange
  var mockMissile = new Mock<OldMissile>();
  mockMissile.Setup(x => x.Fire()).Returns(false);
  var oldLauncher = CreateOldLauncher(mockMissile.Object);
  // act & assert
  Assert.Throws<Exception>(() => oldLauncher.Launch());
  mockMissile.Verify(x => x.Fire());
}
```

Another common anti pattern is to have external calls (to a database, filesystem or network location) directly in the workflow. Again, refactor that code into a separate class (i.e. database calls in [database context scope](http://mehdi.me/ambient-dbcontext-in-ef6/)) and inject their interfaces. That way, the external part of the code can be mocked and thus controlled by the test and the new class has a single responsibility: going to the resource. Which in turn can easily be tested on its own.

## Conclusion

Even simple scenarios like these take a number of changes, more complex scenarios will take more work to be able to test. The most important thing to remember is to never be afraid to change code. If you are, then something is wrong and that should be investigated. Testing older code can be very challenging, but that means it is worthwhile to do so.

In the last post I will bring everything together in a simple console application. If anybody notices any inconsistenties, please contact me on [twitter](https://twitter.com/bonny_ken/) or via [email](mailto:bonny.ken@gmail.com).
