---
title: "Rediscovering implicit casting"
datePublished: Mon Jul 05 2021 08:39:45 GMT+0000 (Coordinated Universal Time)
cuid: ckqqdhau312fss2s1dr5kd11d
slug: rediscovering-implicit-casting-1
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1625403867330/WdtH8yKxY.jpeg
tags: csharp, dotnet, dotnetcore

---

A lot of my ideas are good, but unfortunately not all of them. On [my last blog post](https://kenbonny.net/rediscovering-implicit-casting), I got feedback (via two [reddit](https://www.reddit.com/r/csharp/comments/nv0gqg/rediscovering_implicit_casting/) [posts](https://www.reddit.com/r/dotnet/comments/nv0gwk/rediscovering_implicit_casting/)) that I did not think things through enough. (My wife says that happens more than I realise.) After mulling it over, I realised that the comments that I received are valid and that I need to rectify my mistake.

When I was thinking about ways to make Kotlins type aliases available in C#, I was too focussed on the perceived ease of representing one thing as something else. A `string` as a `Uri` for example. I did not investigate deeply enough to see that Kotlins type aliases are syntactic sugar to make concepts more explicit and not a real conversion.

This brings me to the first comment and I even put it right into my blog post. 🤦‍♂️

> An implicit cast should always succeed and return a result.

This is stated originally in the [Microsoft guidelines](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/user-defined-conversion-operators):

> Predefined C# implicit conversions always succeed and never throw an exception.

Why is it important that there is no exception? Because of the implicit connection between the two types. One type should always be the other type. With an explicit cast, this can be said to be the case sometimes. I'm forcing the cast from one type to another, but it can go wrong. When I'm using an implicit cast, this should be a given.

For example: I can say that a `Uri` can always be represented as a `string`. I cannot say that every `string` should always be a valid `Uri`. Especially not if I say that every `string` should also be a valid `FileInfo` and a valid `DirectoryInfo` and a valid `Telephone` number. A `string` cannot be all at the same time. Yet that is what this implicit cast would indicate.

Therefore, my notion that all their classes should be able to transparently cast from `string` to their type is wrong.

A better way to do this would be to write convenience overloads for methods that take a `Uri` so it accepts a `string`.

```
class DownloadSomething
{
    Task<T> Data<T>(string location)
    {
        // parse string to uri
        Data(new Uri(location));
    }
    Task<T> Data<T>(Uri location)
    {
        // download data
    }
}
```

This makes it easy to pass in a `string` instead of a `Uri`, yet gives me the option to parse the `string` to make sure this is a valid `Uri`.

This brings me to the second point: code should not be magic. If I read the code, I should understand what it is doing, unambiguously. Suddenly going from a `string` to a `Uri`, without checking, verifying or parsing is not always intuitively. In some cases, it might be, but in a lot more, it won't be.

For as long as I've programmed, I've disliked clever tricks or techniques that look like magic. This should have looked like magic as well, but I was staring too blindly at the Kotlin type alias functionality. This clicked when I read this [really good explanation](https://www.reddit.com/r/csharp/comments/nv0gqg/rediscovering_implicit_casting/h12hx7t?utm_source=share&utm_medium=web2x&context=3) about Visual Basics `Nothing` keyword.

To end on a positive note, I think I have found a good place for an implicit cast: inside the builder pattern. A builder should always have a valid instance of the type it has been building. So, casting a builder to the type it's creating is safe, can always be done and does not feel like magic. It works very nicely in my tests where I need to prepare several input objects. For example, an order with several items:

```
database.Add(OrderBuilder.Create(orderDate, personOrdering)
                         .AddItem("Item 1", price:200, quantity:10)
                         .AddItem("Item 2", price:500, quantity:2))

class OrderBuilder
{
    public OrderBuilder Create(DateTime orderDate, Person client)
    {
        // create order
        return this;
    }

    public OrderBuilder AddItem(string description, int price, int quantity)
    {
        // add item to order
        return this;
    }

    public Order Build()
    {
        // return order with lines
    }

    public static implicit operator Order(OrderBuilder builder) => builder.Build();
}
```

Making mistakes is never fun, but I did learn a great deal from it. I learned a lot about casting, I got a history lesson about Visual Basic and I got a lesson in humility. Now I hope others can learn from my mistake as well.

