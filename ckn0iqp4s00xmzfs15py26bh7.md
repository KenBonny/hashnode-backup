---
title: "Clean Architecture Applied"
datePublished: Mon Mar 11 2019 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iqp4s00xmzfs15py26bh7
slug: clean-architecture-applied-the-work-repository
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617801633387/u-5kyX2sJ.jpeg
tags: csharp, software-architecture, dotnetcore

---


Now that I have the general steps to get to an invoice, I can start working on the individual components that reside in the infrastructure layer; starting with the work repository.

The domain logic would need time data, which reminds me of the [Repository pattern](https://martinfowler.com/eaaCatalog/repository.html). So I create an interface `` `IWorkRepository` `` that gets work days between two dates: `Task<WorkDay[]> GetWorkDays(DateTime start, DateTime end)`.

```
[DebuggerDisplay("{DebuggerDisplay}")]
public class WorkDay
{
    public WorkDay(string customerTag, DateTime day, int secondsWorked, bool billable)
    {
         Day = day;
         TimeWorked = TimeSpan.FromSeconds(secondsWorked);
         Billable = billable;
         CustomerTag = customerTag;
    }
    public DateTime Day { get; }
    public TimeSpan TimeWorked { get; }
    public bool Billable { get; }
    public string CustomerTag { get; }
    private string DebuggerDisplay => $"{CustomerTag} - {Day:d}: {TimeWorked}h ";
}
```

The private DebuggerDisplay field is a little workaround to allow easy string manipulation (especially dates) to get nice debug information.

This would allow me to use a fake work repository when testing to start verifying other behaviour. However, I learned about [Flurl](https://flurl.io/), a nice library that allows me to build HTTP requests easily, but also allows me to use it for testing. The next thing I did was build the `TimeCampWorkRepository`, referencing the actual TimeCamp API URL. Flurl allows me to use the `TimeCampWorkRepository` in the tests so I can verify that the JSON I retrieve from the API is correctly deserialised and mapped to the `WorkDay` structure. I got that JSON from actually calling the API and saving the response. This allows me to verify my own code to the point right where the actual API call will happen and how the returned information is handled.

This also ties into the Clean Architecture principle that the database, or more general data store, should be pushed to the infrastructure layer. I put this code in its own project to clearly separate this concern. Putting an interface between the actual call to the TimeCamp API and the control flow of the invoice generator allows me to add other data sources if I should ever change the app I use to track my time.

Next up is the document generator.
