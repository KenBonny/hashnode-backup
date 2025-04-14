---
title: "C# like object initialisation in TypeScript"
datePublished: Mon Oct 29 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iptj800xjzfs1aa9ued3i
slug: c-like-object-initialisation-in-typescript
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380711825/P5TAg9_B1.jpeg
tags: typescript

---


In [TypeScript](https://www.typescriptlang.org), I've missed the ease of initialising objects as I have in dotnet with [object initalisation](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/how-to-initialize-objects-by-using-an-object-initializer). While researching an issue, I stumbled upon a little trick that makes initialising an object a breeze in TypeScript.

Let's dive right in with an example class:

```
class MyClass {
  id: number;
  text: string;
}
```

Now, if I want to initialise this, I have a few options. I can add a constructor, but then I'd need a constructor for every possible combination or a lot of nullable properties.

```
consturctor(id: number, text: string) {
  this.id = id;
  this.text = text;
 }
```

But this leaves me with the problem that I don't always want to set the id. So I'd need to either create a constructor with nullable parameters which leads to _"very clean"_ constructors like `new MyClass(null, 'some text')` or a constructor for every combination that I would like. Which, in my opinion, is very time consuming and I'll always forget one or I'll have to create a new one later for an additional scenario.

An alternative is to just use give each property a public setter so I can initialise the properties after I create the object.

```
const myClass = new MyClass();
myClass.text = 'some text';
```

Why even bother with all the benefits of immutability or hiding certain properties from the public.

To solve this in a reusable way, let's get the `Partial<>` class involved to solve this little conundrum. This class can hold part of a strongly typed object. So I can have auto completion support, yet I don't have to supply all properties. This allows me to create a single constructor with an optional `Partial<MyClass>` parameter that can take any combination of properties.

```
constructor(init?: Partial) {
  Object.assign(this, init);
}
```

The `Object.assign(this, init);` maps all the properties in the `Partial&lt;&gt;` object to the public available properties of `this` object. This works fine with both fields and properties.

Now I can use all of the following constructors.

```
new MyClass();
new MyClass({text: 'some text'});
new MyClass({id: 1, text: 'some text'});
```

That's a pretty nice feature to have in my code.
