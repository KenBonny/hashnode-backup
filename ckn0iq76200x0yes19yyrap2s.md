---
title: "Clean Architecture Applied"
datePublished: Mon Apr 08 2019 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iq76200x0yes19yyrap2s
slug: clean-architecture-applied-bringing-it-all-together
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617801380637/GF4tJiYxL.jpeg
tags: csharp, software-architecture, dotnetcore

---


Now all individual components are ready, it's time to bring it all together in a working application.

To get all these parts together, I add the last project: the console that will run the whole thing. It wouldn't be complete with the popular syntax of `invoice auto`. The [Command Line Utils](https://github.com/natemcmaster/CommandLineUtils) package allows me to convert the `invoice auto args` from the `Main` method into commands that can be executed. This package has a bit of a learning curve, but has a lot of options to customise the command line arguments. I'm not going to elaborate a lot on that, I liked the package and I'll let my readers figure out how to use it best.

Lastly, there's the setup of the objects in the Main function. It's basic injection of the services with a little composition for the `pdfGenerator` that takes the `htmlGenerator` as input.

```
// in my console application
var dateProvider = new SystemDateProvider();
var customerRepository = new StaticCustomerRepository();
var timeCampWorkRepository = new TimeCampWorkRepository(configuration, customerRepository);
var htmlGenerator = new FluidHtmlDocumentGenerator();
var pdfGenerator = new DinkToPdfDocumentGenerator(htmlGenerator);
var generator = new InvoiceGenerator(dateProvider,
                                     customerRepository,
                                     timeCampWorkRepository,
                                     pdfGenerator);
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380726316/cTC9Y_1U2.jpeg)

#### Structuring the code

The `ICustomerRepository` interface should be placed in the use case layer (orange) and the implementation should be placed in the infrastructure layer (blue). Clean Architecture suggests grouping things that change with the same frequency and for the same reason. The infrastructure layer normally changes a lot more than the use case layer. I put the `WorkRepository` and the `StaticCustomerRepository` in separate projects. They change for different reasons at different times, so I should be able to build and deploy them at different times.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380727908/vAEtI7OWY.jpeg)

The document generators (`FluidHtmlDocumentGenerator` and `DinkToPdfDocumentGenerator`) are in a different project together. They both change when I need to update the document generation. Maybe not at the exact same time, but definitely for the same reason.

Above all else, the code should be easily testable. That is why I used all the interfaces. It's easy to switch out an actual component for a fake one. The fake implementation allows me to control certain data. For example: I have a `IDateProvider` interface so I can control when `Now` actually is. If I would just use `DateTime.Now`, I could not simulate generating an invoice for a specific date. Now I just have two implementations: the `SystemDateProvider` in my actual application and the `FakeDateProvider` for in my tests.

```
// shared interface
public interface IDateProvider
{
  DateTime Now { get; }
}

// in my console application
internal class SystemDateProvider : IDateProvider
{
  public DateTime Now => DateTime.Now;
}

// in my tests
internal class FakeDateProvider : IDateProvider
{
  public FakeDateProvider(string now)
  {
    if (DateTime.TryParse(now, out var parsedNow))
    {
      Now = parsedNow;   
    }
  }
  public DateTime Now { get; }
}
```

The console project belongs in the infrastructure layer, together with the implementations of the interfaces. I put it in another layer in the image to indicate that the console brings all the other layers together.

The last blog in this series will talk about the pros and cons of this architecture.
