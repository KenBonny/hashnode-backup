---
title: "Constructor fun"
datePublished: Mon Jan 08 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0irshb00xuzfs1ebbm2q1t
slug: constructor-fun
tags: csharp, dotnet, dotnetcore

---


At work, I have a small gripe about a `Response` class. It's minor, but it keeps bugging me.

The object is very simple, it returns either a generic value or an error message and a Boolean to determine which was returned. It's a very simple object. My gripe is that there is no constructor to distinguish between a value or error returned. Every time I need to instantiate a `Response` object, I need to set the `IsError` property.

So I added two constructors, one for a value and one for an error.

```
public class Response<T>
{
  public Response(T value)
  {
    Value = value;
    IsError = false;
  }
  public Response(string error)
  {
    Error = error;
    IsError = true;
  }
  public T Value { get; }
  public string Error { get; }
  public bool IsError { get; }
}
```

Only when I started using this, I noticed a problem. What if the value I want to return is a `string`. With other objects, there is no conflict, but with a `string` value, the constructor thinks it is an error that is passed along.

Fortunately, there is a very simple solution. When I initialise the `Response`, I can select the constructor to use by using the parameter name.

```
var text = "no error";
var errorResponse = new ResponseMessage(text); // IsError = true
var valueResponse = new ResponseMessage(value: text); // IsError = false
```

In the actual code, I left in an empty constructor, so older code compiles.

The lesson I learned here is to think how I will use the classes I create and guide the instantiation process. It will help me down the line. Besides, fixing small gripes like this are quick wins that make me feel better about the code I produce.

All code can be found in [this gist](https://gist.github.com/KenBonny/be5add0a438ede0eb1386e93657c79ff).

_UPDATE_: A friend, [Wesley Cabus](https://wesleycabus.be), took this idea a bit further and thought about how to do this a little cleaner. The problem he sees, is that it becomes difficult in a `string` case to distinguish between a value (`new Response(value: "value")`) and an error (`new Response("error")`) and I fully agree. So if you need a solution to this problem and are not tied to an existing solution, go check out [Wesleys blogpost](https://wesleycabus.be/2018/01/a-way-to-deal-with-HTTP-error-responses/).
