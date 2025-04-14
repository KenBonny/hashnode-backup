---
title: "Patterns and disciplines from a proof of concept"
datePublished: Mon Aug 14 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iyzy500zhyes16jggf57d
slug: patterns-and-disciplines-from-a-proof-of-concept-part-1
tags: design-patterns, csharp, dotnet, dotnetcore

---


_While I worked at my previous employer, I build a proof of concept to improve their ability to search. I will rebuild that proof of concept and I'll highlight all the patterns and principles I used to build this code. All code related to this proof of concept can be found in a [repository](https://github.com/KenBonny/KenBonny.Search) on [my Github account](https://github.com/KenBonny)._

In this first part, I shall describe the problem space and how the rest of the application interacts with the search functionality.

A quick word on the problem space first. I'll build search functionality that will find seats for diners at restaurants. Different searches can have different needs, so the search should be flexible to allow for different search criteria. I'm not taking dates for the reservations into consideration as this adds a layer of complexity to the model and I want to focus on technical patterns.

There might be some weird use cases in this proof of concept, that is because I'm working with a substituted conceptual model. The original model contains too much specific terminology that I would need to explain and it contains company specific information I imagine ex-colleagues would rather keep in-house.

Lets start with how the rest of the application will interact with the search: through the `ISeatSearcher` interface. Everywhere a search for a seat needs to happen, this interface should be injected. This allows to have several different implementations depending on what is needed. The first thing I want to draw attention to is the descriptive name. Never give bland names, always be as descriptive as you can be. Use jargon where appropriate to shorten names so you don't end up with a whole book for a name.

```
public interface ISearcher
{
  IReadOnlyCollection<Seat> FindSeats(SearchQuery query);
}
```

The return object is a simple representation of a complex domain. This is to hide all the complexity that is needed in one part of the application from another part. Not every part of the application needs to know all the rules in the domain model. Sometimes the rules can differ depending on the situation, for example in read or write scenarios.

```
public class Seat
{
  public Seat(string restaurant, int sectionId, int tableId)
  {
    Restaurant = restaurant;
    SectionId = sectionId;
    TableId = tableId;
  }
  public int TableId { get; }
  public int SectionId { get; }
  public string Restaurant { get; }
}
```

The input object is inspired by the [CQRS design pattern](https://martinfowler.com/bliki/CQRS.html), more specifically the query part. Since it's the intention of any `ISeatSearcher` implementation to only search and never update any records. The `SearchQuery` is a base class that contains only the most basic information that every query would need. Every specific query will get it's own subclass with the specific information that query needs.

For example: I want to find a seat for a diner, then I use the `UnreservedSeatForDinerQuery` that takes the first and last name of the diner. The specific implementation will know what to do with the name of the diner, such as look for a table where one of the guests shares a last name so family can be seated together.

```
public class SearchQuery
{
  public SearchQuery(SortOrder sortOrder)
  {
    SortOrder = sortOrder;
  }
  public SortOrder SortOrder { get; }
}
public enum SortOrder { BestFirst, WorstFirst }
public class UnreservedSeatForDinerQuery : SearchQuery
{
  public UnreservedSeatForDinerQuery(string dinerFirstName, string dinerLastName, SortOrder sortOrder = SortOrder.BestFirst) : base(sortOrder)
  {
    DinerFirstName = dinerFirstName;
    DinerLastName = dinerLastName;
  }
  public string DinerFirstName { get; }
  public string DinerLastName { get; }
}
```

All this code can be found in the _Core_ project. Note that there is no implementation within this project as I want to keep all implementations in their respective projects. There is one exception for the mediator pattern, but I will come back to that in a later part.

In the next part, I will go into the details of the read model for the default implementation of the search algorithm.
