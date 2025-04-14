---
title: "Creating a custom xUnit [Trait]"
datePublished: Tue Dec 20 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0is8ok00y0zfs1gy2g5tcs
slug: creating-a-custom-xunit-trait
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1618320398012/6BrKXdNO_.png
tags: csharp, unit-testing, testing, dotnet, dotnetcore

---


After creating my blog posts on [writing tests with xUnit in a TDD fashion](https://kenbonny.net/tag/better-code-with-tests/), I remembered I had only briefly touched upon `Trait`s and not at all how to implement custom ones. Time to remedy that.

There is not a lot I can say more about traits than I did in my [original post](http://kenbonny.net/2016/09/21/unit-testing-part-4/), all the way at the bottom.

> With the `[Trait]` attribute, I can add metadata to tests. The first parameter is the key of the metadata, the second parameter is the value that goes with the key. This can be a bit confusing when starting to work with the xUnit framework. I like to think about traits as a sort of claim from [claims-based authentication](https://msdn.microsoft.com/en-us/library/ff359101.aspx).

What I can do, is elaborate on creating custom attributes. My original information came from [the only blog post I found that described xUnits new way of extending Traits](http://mac-blog.org.ua/xunit-category-trait/). I think that counting constructor parameters and tight coupling of `Trait`s and `TraitDiscoverer`s is not the most effective way of handling `Trait`s. Here is hopefully a better way. All code is available on my [GitHub account](https://github.com/KenBonny/CustomXunitTrait), the traits can be found in _CustomXunitTrait/CustomXunitTrait.Tests/Infrastructure/_.

## Simple Trait

A simple `Trait` is an attribute without derived classes. It stands on its own. For example: an analyst on the team has described a lot of the functionality of the application in test cases. I as a developer want to add the test case identifier to the appropriate unit test. This allows the documentation of the feature to be quickly found in the analysis. Take a look at the `Trait` below to see what I mean.

```
[TraitDiscoverer("CustomXunitTrait.Tests.Infrastructure.TestCaseDiscoverer", "CustomXunitTrait.Tests")]
[AttributeUsage(AttributeTargets.Method)]
public class TestCaseAttribute : Attribute, ITraitAttribute
{
  public string TestCase { get; set; }
  public TestCaseAttribute(string testCase)
  {
    TestCase = testCase;
  }
}
```

With a little workaround, the discoverer can easily use the properties of the custom `Trait`.

```
public class TestCaseDiscoverer : ITraitDiscoverer
{
  private const string Key = "TestCase";
  public IEnumerable<KeyValuePair<string, string>> GetTraits(IAttributeInfo traitAttribute)
  {
    string testCase;
    var attributeInfo = traitAttribute as ReflectionAttributeInfo;
    var testCaseAttribute = attributeInfo?.Attribute as TestCaseAttribute;
    if (testCaseAttribute != null)
    {
      testCase = testCaseAttribute.TestCase;
    }
    else
    {
      var constructorArguments = traitAttribute.GetConstructorArguments().ToArray();
      testCase = constructorArguments[0].ToString();
    }
    yield return new KeyValuePair<string, string>(Key, testCase);
  }
}
```

To access the attribute, I cast the `IAttributeInfo` object to a `ReflectionAttributeInfo` object which allows me to access the actual attribute that is being parsed. The property contains an object, since I don't see an easy way to couple the `ITraitDiscoverer` to an instance of `ITraitAttribute`. If you wonder why the discoverer can't accept a generic type that implements `ITraitAttribute`, I suggest you read my **Points of improvements** paragraph first.

Now that I can access the attribute, I can easily get the values that were passed along to it and saved in its properties. This is a lot clearer to use than the `GetConstructorArguments()` that is used in the `else` part of the code. I inserted that as a fallback when the casts should fail. Failing on the cast can also be interesting because it would force you to investigate the problem.

## Complex Trait

A complex `Trait` is an attribute that has derived classes for every possible option I want to make available. For example: I build an abstract `CategoryAttribute` to allow categories to be specified and then provide derived `ComponentCategoryAttribute` and `EndToEndCategoryAttribute` to indicate which tests are component or end-to-end tests.

```
[TraitDiscoverer("CustomXunitTrait.Tests.Infrastructure.CategoryDiscoverer", "CustomXunitTrait.Tests")]
[AttributeUsage(AttributeTargets.Method)]
public abstract class CategoryAttribute : Attribute, ITraitAttribute
{
  public abstract string Type { get; }
}
public class ComponentCategoryAttribute : CategoryAttribute
{
  public override string Type => "Component";
}
public class EndToEndCategoryAttribute : CategoryAttribute
{
  public override string Type => "EndToEnd";
}
```

This now allows me to write a very easy to understand `ITraitDiscoverer` that can handle both categories.

```
public class CategoryDiscoverer : ITraitDiscoverer
{
  private const string Key = "Category";
  public IEnumerable<KeyValuePair<string, string>> GetTraits(IAttributeInfo traitAttribute)
  {
    var attributeInfo = traitAttribute as ReflectionAttributeInfo;
    var category = attributeInfo?.Attribute as CategoryAttribute;
    var value = category?.Type ?? string.Empty;
    yield return new KeyValuePair<string, string>(Key, value);
  }
}
```

Look at how clean this is solved now. I cast the `IAttributeInfo` object to a `ReflectionAttributeInfo` instance, I cast the `Attribute` property to the `CategoryAttribute` class and I can access the value stored inside to indicate the correct category of the test.

## How to use these Traits

```
[Fact]
[TestCase("Biz001")]
[ComponentCategory]
public void FirstTest()
{...}
[Fact]
[TestCase("Biz002")]
[EndToEndCategory]
public void SecondTest()
{...}
```

Unfortunately, at the time of writing, the custom categories are pretty useless since there is [no immediate support for them in the ReSharper test discoverer](https://github.com/xunit/resharper-xunit/issues/35). They can be convenient to indicate the purpose of the test.

## Possible improvements

There are two issues with the implementation that I would like to see changed.

First, the `ITraitDiscoverer` passes an `IAttributeInfo` object to the `GetTraits` method. The `IAttributeInfo` should contain the `Attribute` property without having to cast it to a specific type. The `Attribute` property would of course need a cast, but not the object passed to the method. I think the `Attribute` property is a pretty important object to pass to the discoverer, who knows what people would want to do with it to construct the `IEnumerable&lt;KeyValuePair&lt;string, string&gt;&gt;` return type.

The other improvement is on the attribute itself. The problem I have is that the `TraitDiscoverer` attribute that is applied to a custom `Trait` class needs a type and assembly name as a `string` to locate the `ITraitDiscoverer`. If I refactor the name, namespace or assembly, then I have to change the text in all the attributes I created. I wonder why tests can't see if an attribute implements the `ITraitAttribute`. The `ITraitAttribute` could then accept a generic type that expects an `ITraitDiscoverer` to be passed along.  Example:

```
[AttributeUsage(AttributeTargets.Method)]
public class TestCaseAttribute : Attribute, ITraitAttribute<TestCaseDiscoverer>
{...}
```

This would mean that the `ITraitDiscoverer` could not be made generic, as this would create a circular reference if the attributes and discoverers would be located in two separate assemblies.
