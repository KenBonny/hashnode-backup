---
title: "Clean Architecture Applied"
datePublished: Mon Mar 04 2019 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iq13400wzyes1ggri4yiz
slug: clean-architecture-applied-applying-the-principles
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380721016/xIlhAVVW2.jpeg
tags: csharp, software-architecture, dotnetcore

---


Before I dive into the technical details, let me give you a little warning:  

> This solution is over-engineered, I could write this fairly straight forward and be done in a few hours. The experiment here is to implement this app using the guidelines from Clean Architecture. So expect too many projects, a bit of plumbing code and a lot of interfaces.  

> Also, I cannot make the full code available as this code contains secrets (such as my customers). It would also allow anybody to generate invoices in my company's name. I will not be making this code easily available. Ever.  
I thought about making an example repository to share with the world. I do not think I would give that repository the same attention I give this code, it would not represent the quality that I put into my work.  
If you or your company need help with improving code quality, I offer consultancy services and training. Head over to [More Than Code](https://www.morethancode.be) for more information.

The problem domain is not a difficult problem, so let's check out how I structured the code to get it all working.

The first step is to get the data from the TimeCamp API, which I need to transform into days worked so I can sum them and then put them into a format that I can save as a PDF-file.

I started by creating a solution with a project _InvoiceGenerator.Core_. I can also name it .Business, .Domain or .Rules, pick something that clearly communicates what is in the project. At the same time, I create an _InvoiceGenerator.Tests_ project, because all important parts need to be tested well.

Because this is not a large project, I only create one test project. I believe in testing use cases, such as "generate HTML invoice from TimeCamp data", that test a lot my own code (I'll get back to that in a minute). This allows me to change the inner working of a module, without losing any functionality. My tests will guard the functionality. That is why my tests will always be very specific. This test knows about the TimeCamp API and that the invoice will be in the HTML-format.

What I mean by "my own code" is all code that I write. It encompasses everything from domain logic, internal plumbing and all code right up to the point it goes out of my hands, such as to a database, the network or the hard drive.

![Domain and Use Case](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380719613/GnoeMnlKR.jpeg)

#### Invoice generator

The first test then sounds easy: generate an HTML invoice from TimeCamp data. It all starts in the core business logic. For that I create a class `InvoiceGenerator` in the _Core_ project. This will orchestrate the interactions between all the little components. It determines the month the invoice should be generated for (the previous one). Then it gets the data about the days I worked. Finally, it generates invoices for all customers I worked for.

```
public async Task<Invoice[]> Generate()
{
  var startOfPreviousMonth = _dateProvider.Now.StartOfPreviousMonth();
  var endOfPreviousMonth = startOfPreviousMonth.AddMonths(1).AddDays(-1);
  var workDays = await _workHoursRepository.GetWorkDays(startOfPreviousMonth, endOfPreviousMonth);
  var billableWorkDays = workDays.Where(day => day.Billable);
  var invoices = new List<Invoice>();
  var previousInvoiceFileName = _invoiceReader.GetPreviousInvoice();
  var description = _dateProvider.Now.ToString("MMMM").ToLower();
  foreach (var daysForCustomer in billableWorkDays.GroupBy(x => x.CustomerTag))
  {
    var customerTag = daysForCustomer.Key;
    var customer = GetCustomer(customerTag);
    var billableItems = GetBillableItems(customer, daysForCustomer, startOfPreviousMonth);
    var invoice = GeneratedInvoice(previousInvoiceFileName, customer, description, billableItems);
    previousInvoiceFileName = invoice.FileName;
    invoices.Add(invoice);
  }
  return invoices.ToArray();
}
```

The `_dateProvider` is there to make injecting a fake date a lot easier. It's quite difficult, read impossible, to change the `DateTime.Now` date. Which makes testing different months a lot easier.

The previous invoice is used to calculate the new invoice number, this ensures that I can keep numbers nice and sequential.

#### Adding immutable objects

Dealing with state is complex. Most bugs are introduced when unexpected data influences the flow of code. That is why I made most objects that pass through boundaries [immutable](https://en.wikipedia.org/wiki/Immutable_object). Objects from the `InvoiceGenerator` back to the console (which will become my UI, I will go into more detail in a future post on this aspect) or from the `DocumentGenerator` back to the `InvoiceGenerator` are being created by the originating code (`InvoiceGenerator`, `DocumentGenerator`).

```
public class Invoice
{
  private readonly string _description;
  private string _extension;
  private readonly int _id;
  private Invoice(int id, DateTime invoiceDate, string description, byte[] document)
  {
    _id = id;
    _description = description;
    InvoiceDate = invoiceDate;
    Document = document;
  }

  public byte[] Document { get; private set; }
  public DateTime InvoiceDate { get; }
  public DateTime ExpiryDate => InvoiceDate.AddMonths(1);
  public string Number => $"{InvoiceDate:yyyy}{_id:0000}";
  public string FileName => $"{Number}-invoice-{_description}.{_extension}";
  public static Invoice FromPrevious(DateTime invoiceDate, Customer customer, string description, byte[] document)
  {
    var id = GetId(invoiceDate);
    return new Invoice(id, invoiceDate, description, document);
  }
}
```

This way the document or invoice can't be updated after it's created. Not by my or any other code. I make it easy on myself by creating a [static factory method](https://stackify.com/static-factory-methods/) that can take care of creating an object. I could just use the constructor, but I think the method is a bit cleaner to read.

#### The principles

The core domain consists of the general flow of the application. Getting workdays, grouping them per customer, transforming them to billable records and filling that data into invoice templates.

It then hands control to several plugins (such as a `IWorkRepository` or a `DocumentGenerator`). This is part of the layer approach. It also allows me to experiment with different ways of getting the workdays and different template engines to generate an invoice. It also means that I can easily test different components.

I had already thought about how I would tackle each issue, but let's go through it step by step. Starting with the work repository.
