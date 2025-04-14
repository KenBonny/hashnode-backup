---
title: "Naming is hard"
datePublished: Mon Jun 12 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0ixtal00zuzfs1a6zb3yl9
slug: naming-is-hard
tags: programming, naming

---


Especially when programmers don't agree on how variables and classes should be named.

Vladimir Khorikov published an article about [Ubiquitous Language and Naming](http://enterprisecraftsmanship.com/2017/06/07/naming-and-ubiquitous-language/) in which he explains why a good name is important and how to avoid bad names. I agree with him on about 90% of the content. However, at the end of the article are 2 sentences to which I fully disagree:

> The fact that the repository uses a SQL database as an underlying storage is an implementation detail. No need to manifest it in the class name.

What Vladimir is saying is that if an interface, for example `IUserRepository`, only has one implementation, then a generic name is appropriate. For example: `UserRepository`. When there are multiple implementations, then more specific names should be chosen. For example `SqlUserRepository`, `OracleUserRepository`, `EventStoreUserRepository`, etc.

The reason I disagree is very simple, an interface should communicate the purpose that needs to be fulfilled while the implementation should reveal how that purpose is reached. For example, the interface `IUserRepository` lets me know that via this interface I can access user information. Or the interface `IPaymentProcessor` is used for processing payments.

The implementation of an interface should, in my opinion, reveal how this is achieved. For example, the interface `IUserRepository` could be implemented as `SqlUserRepository` so I know that this implementation targets a SQL implementation. The earlier example `IPaymentProcessor` is a bit trickier because it needs specific business information to figure out a good name for the implementation. One implementation could be `VatIncludedPaymentProcessor`, another one could be `PayPalPaymentProcessor`.

Vladimir recommends only to use a specific name when there are multiple implementations. I recommend always giving a specific name that reveals how an interface is implemented. When I see a `UserRepository` or a `PaymentProcessor`, I'm curious how it accomplishes its task. I have to investigate and find out. When I see a specific name (and I trust the code base), I don't need to investigate. The name tells me all I need to know.

When choosing a class name, look for a name that conveys meaning and reveals implementation details. This way, it will be easy to know how something was done and whether the implementation is still valid in the situation you want to use it.
