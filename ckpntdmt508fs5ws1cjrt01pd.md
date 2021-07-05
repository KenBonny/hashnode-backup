## Rediscovering implicit casting

> ### ⚠ WARNING ⚠
> The information in this blog post is largely wrong. [Read more about it in this blog post](https://kenbonny.net/rediscovering-implicit-casting-1).

With all the [new goodies of C# 10](https://kenbonny.net/introducing-csharp-10) coming soon,  a co-worker noted that [type aliases](https://www.baeldung.com/kotlin/type-aliases) aren't part of this release. He's kinda jealous that [Kotlin](https://kotlinlang.org/) has them and us C# developers *have nothing that comes even remotely close* (loose translation from Dutch). Well, joke's on him, there are implicit conversions in the C# language!

Implicit conversions are pretty cool and I'm kind of bummed that not more of the dotnet framework makes use of them. There will probably be a good reason for it... or they haven't gotten around to adding it yet. But I better not get ahead of myself, so let's start with the basics.

## What is an implicit cast?

There are a number of names for implicit casting such as implicit conversion, implicit type coercion and implicit type juggling. [Wikipedia](https://en.wikipedia.org/wiki/Type_conversion#Implicit_type_conversion) describes implicit casting as:

> Implicit type conversion, also known as coercion, is an automatic type conversion by the compiler. Some programming languages allow compilers to provide coercion; others require it.

This is the *magic* behind lines such as:
 
```
int number = 5;
double dbl = number;
decimal dec = number;
```

This is in contrast to an explicit cast, mostly referred to as a cast or conversion or type coercion, where the type needs to be explicitly specified.

```
float flt = (float) dec;
```

The [guideline](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/user-defined-conversion-operators) whether a cast should be defined as implicit or explicit is that an explicit cast can fail and thus throw an exception or could lose information. So never forget to put a try-catch block around explicit conversions. An implicit cast should always succeed and return a result. I'll come back to this later.

## How are implicit casts specified

To make it all work, the C# spec has a [few keywords to make casting work](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/user-defined-conversion-operators), whether explicit or implicit. It starts with the `operator` keyword. The base use of this keyword can be used to override or add [operations](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/addition-operator) to classes. An example is the + operation on `string`.

To override or add casting, you need to follow a specific pattern: `public static [implicit|explicit] operator ToType(FromType value) { /* conversion code goes here */ }`

So to create an implicit cast from `int` to `double`, I would write: `public static implicit operator Double(int value) { /* convert int to double here */ }`

All operators need to be static as they aren't used on instances of an object, but are used as a sort of extension method. They were available before extension methods, so it's not entirely true. This simile helps me understand how to use them.

Another benefit of these operators being static is that they cannot use or influence non-public fields or properties, which makes them all [pure functions](https://en.wikipedia.org/wiki/Pure_function). Even if I could add side effects in any way, I do not recommend doing that as it violates my principle that there should be no magic involved in programming.

Programming is hard enough as it is, without involving the arcane arts. 😄

## When is it useful to add implicit casts?

I've encountered a number of scenarios where implicit casts would be so convenient. On the top of my head, I'm thinking `string` to `Uri`, `FileInfo` or `DirectoryInfo`. In a lot of cases, I know which `string` refers to a url, file or directory. I would need to do `new Uri(uriAsString)` before being able to use a `string` as a `Uri`. It would allow me to create a method to fetch data from a url and pass in the `string` representation.

```
private T FetchData<T>(Uri location)
{
  // fetch data here
}

// call the method
var data = FetchData<Data>("https://path/to/resource");
```

Casting a `string` to one of the mentioned types cannot fail unless I pass a `null` value, which I don't think happens a lot in these cases. In theory, I would need to make this cast explicit. I also think that there are a lot of types that do this null checking. Therefor I do not see casting a `null` value and getting an error as a valid use case to say that this would need to be an explicit cast.

My practical side says that the benefits of an implicit cast would outweigh the downsides for these types. This is why it would be nice if these types could add implicit conversions from `string`. P.S. don't forget to check whether a `Uri`, `FileInfo` or `DirectoryInfo` is actually a valid representation. I'm talking about handling error codes or checking `.Exists` before using it.

## How to add implicit conversion

At the start of the article, I was talking about type aliases. I could, with a little bit more work than a Kotlin type alias, create similar code. Let's say that I have a type that is represented a lot as a piece of text, but would be nice to know what the intent behind it is. I'm going to take a phone number as an example, but it could be an email address, an address (physical, ipv4, ipv6,...) or anything in the domain at hand.

```
class Telephone
{
  public string Number { get; }

  public Telephone(string number) => Number = number;
  public override string ToString() => Number;
  public static implicit operator Telephone(string number) => new(number);
  public static implicit operator string(Telephone number) => number.ToString();
}
```

Here I've got a `Telephone` class, which has a `Number` property where I'm going to store the phone number. It also has a constructor to set the number and I've overwritten the `ToString` method so I can convert back to `string` easily.

To implicitly convert from a `string` to a `Telephone` object, I've added the line `public static implicit operator Telephone(string number) => new(number);`. It takes a `string` and returns a `Telephone` object. To provide a cast from a `Telephone` back to a string, without calling `.ToString()`, I've added the line `public static implicit operator string(Telephone number) => number.ToString();`. Technically, I am calling the `.ToString()` method, but the compiler takes care of that for me.

With this in place, I can do a lot of pretty cool things.

```
public bool IsValidTelephoneNumber(Telephone number)
{
  // validation logic goes here
}

// use the method
var result = IsValidTelephoneNumber("+32 123 456 789");
```

Let's go a little bit deeper than that. JSON deserialisation works out of the box. Say that I have a `Person` class with both a `Name` and `Telephone` property, then the expected JSON is

```
class Person
{
  public string Name { get; set; }
  public Telephone Telephone { get; set; }
}
// serialised to json
{
  "Name": "Ken",
  "Telephone": {
    "Number": "+32 1234 567 890"
  }
}
```

With the implicit conversion in place, I can actually do deserialisation with JSON.NET of the following object out of the box. While I haven't tested it, my intuition says that an ASP.NET endpoint can receive the following JSON and deserialise it to a correct `Person` object.

```
{
  "Name": "Ken",
  "Telephone": "+32 1234 567 890"
}
```

All of the above code can be found in [this gist](https://gist.github.com/KenBonny/de398822839c8f01409f7f7d935b1cd3). It contains a number of unit tests (written with [xUnit](https://xunit.net/) and asserted with [Shouldly](https://github.com/shouldly/shouldly)), so anybody can verify that this works. It even contains a custom `JsonConverter` to serialise a `Telephone` object back to the second serialisation output.

If `Url`, `FileInfo` or `DirectoryInfo` supported implicit casting from `string`, I could add these types to my configuration class definitions and load them directly from *appsettings.json*. No awkward conversions necessary.

## Where I would not use it

Now that I've demonstrated how easy it is to add implicit (or explicit) casting to an object, there are a few situations where I would not use it.

I think `Url`, `FileInfo` and `DirectoryInfo` could really benefit from implicit casting because they represent a specific use case of a `string`. So what's stopping me from adding `DateTimeOffset` to the list? The number of parsing options.

A `string` can be a url and there are only a few ways to parse it. There are no `TryParse` methods on `Uri` (or any of the other examples). There are also only a few options when parsing a url or a path to a file.

In contrast, there are quite a number of ways to parse a `string` into a `DateTime` and never mind the number of added options when I use a `DateTimeOffset`. I can't just convert a `string` to a `DateTimeOffset` without specifying a number of options such as format, culture or time zone. I could take educated guesses based on the computer settings (there is a way to `TryParse` without those options), but with implicit or explicit conversion there is no way to specify those options.

The `TryParse` method also allows for failure, it tells me if I specified a valid date. I could use the `Parse` method and it would throw an exception if the date could not be parsed. Then I would need to add try-catch blocks, just to safely cast a `string` to a `DateTimeOffset`. I would never need to do that to a `Uri` or `FileInfo` or `DirectoryInfo`. I also hope it's general knowledge by now that [using exceptions for flow control is a bad idea](https://www.google.com/search?q=don%27t+use+exceptions+for+flow+control).

What I find a good middle ground is that the cast of `string` to `DateTimeOffset` would be an explicit cast as there is a good chance an exception will be thrown. I do not have this concern when trying to convert a `string` to a `Uri`, `FileInfo` or `DirectoryInfo`.

This also brings me to my next part: validation. I would rather build an explicit validation engine to verify that a `Telephone` is valid. I would add explicit rules such as `StartsWithCountryCodeOrZero` and `HasXNumberOfDigits`. I would not put all that logic into the constructor.

Adding it to the constructor would hide all the validation logic and would not provide a result with a reason why validation would fail. I would need to expose that on the `Telephone` object and it would not serve a single purpose anymore. I could throw an error, but then every cast would need to be wrapped in a try-catch block and I've touched on this problem earlier.

This approach would also violate the [single responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle) as the `Telephone` class now has multiple reasons to change. It would make the code harder to understand as well. So there are a number of downsides to this approach of validation.

## Conclusion

This might not be as succinct as Kotlins type aliases, but I think C# programmers have a lot of options creating types that represent concepts. It's not that hard to implement patterns that easily convert `string`s to more specific types such as `Telephone`, `Email` and hopefully some framework specific classes as well in the future.