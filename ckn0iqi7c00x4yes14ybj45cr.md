---
title: "Clean Architecture Applied"
datePublished: Mon Mar 25 2019 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iqi7c00x4yes14ybj45cr
slug: clean-architecture-applied-the-customer-repository
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617801499433/xU9sliepL.jpeg
tags: csharp, software-architecture, dotnetcore

---


There are a lot of places where I need customer information. In the TimeCamp repository I need to link a work item to a customer. In the document generator, I need customer information such as name, address and VAT number.

Assigning each work item to a specific customer would take a lot of time, but I know that each item starts with one of two specific prefixes (WRK-001 or BUG-001 to indicate a story or a bug). My customer object will have a list of prefixes so I can identify which customer has to pay what amount.

I don't want to add this information to the configuration of the app. If I ever want to move this logic to, let's say, an Azure function, I don't want to duplicate the customer setup. Setting up a proper database is too much work for now (not to mention a little costly to just store just a few records).

The easiest solution is to add a hard-coded class with customer information. To make sharing easier, I put the `ICustomerRepository` in a shared library and the implementation in another library. I created a fluent builder class so it's easy to set up a new customer. I think it is a lot more descriptive than have a `new Customer {Name = "My Customer NV"}`. Opinions may vary on this topic.

```
public class StaticCustomerRepository : ICustomerRepository
  {
    public Customer[] GetAll()
    {
       return new[]
       {
          MyCustomer,
       };
    }    
    private static Customer MyCustomer => CustomerBuilder.NewCustomer.WithTagAndName("my-customer", "My Customer NV")
      .WithVatNumber("VAT1234567890")               
      .ChargingDaily(100)
      .WithIdentifiers("WRK-", "BUG-")
      .AtAddress("Address", "Info")
      .Build();
}
```

The share library resides in my use case layer where the other interfaces (`IWorkRepository` and `IDocumentGenerator`) live. I put this in a shared project so I could use this in another, separate, project that generates timesheets for my customers that uses the same customer information. I won't be going into detail about that as that would lead us too far off track. I hope it won't be too hard to imagine how that would work, using the principles of Clean Architecture.

Now that all components are finished, I have to bring it all together in a working application.
