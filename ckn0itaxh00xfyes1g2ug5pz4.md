---
title: "Easily compare complex expected output with actual output"
datePublished: Mon Oct 08 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0itaxh00xfyes1g2ug5pz4
slug: easily-compare-complex-expected-output-with-actual-output
tags: csharp, unit-testing, unit-tests, dotnet, dotnetcore

---


A coworker showed me this "little trick which will make your code a lot easier". Ok, I promise I won't use click-bait anymore. The library he showed me, did make testing differences in output a lot cleaner though.

The tests we were writing, were to verify [AutoMapper](https://automapper.org/) mappings. The tests are not only there to ensure that all properties get mapped correctly. But also that when properties get added or removed, they are properly handled in all mapping scenarios. Don't limit yourself to just using this for mapping tests. I would use this for [Golden Master testing](https://en.wikipedia.org/wiki/Characterization_test), verifying generated json, html, pictures, pdfs and probably even more things.

For simplicity's sake, I'll use a mapping example to show how this works. Say that I have two classes: a `Person` class that models a person in the database and an `Employee` class that models an employee from my domain model.

```
namespace KenBonny.Entities
{
    public class Person
    {
        public int Id { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public DateTime EmployedSince { get; set; }
    }
}
namespace KenBonny.Domain
{
    public class Employee
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public DateTime EmployedSince { get; set; }
    }
}
```

What I need next is a `Profile` for AutoMapper.

```
class MappingProfile : Profile
{
    public MappingProfile()
    {
        CreateMap()
            .ForMember(d => d.Name, m => m.MapFrom(s => $"{s.FirstName} {s.LastName}"));
    }
}
```

The mapping from `FirstName` and `LastName` to `Name` is there to ensure that something else besides copying properties into a new object happens.

The last step is to verify the behaviour with a test.

```
[Fact]
public void TestMapping()
{
    var configuration = new MapperConfiguration(cfg => cfg.AddProfile(new MappingProfile()));
    IMapper mapper = new Mapper(configuration);
    var person = new Person
    {
        Id = 1,
        FirstName = "Ken",
        LastName = "Bonny",
        EmployedSince = new DateTime(2018, 1, 1, 11, 00, 00)
    };
    var employee = mapper.Map(person);
    var employeeJson = JsonConvert.SerializeObject(employee);
    ApprovalTests.Approvals.VerifyJson(employeeJson);
}
```

I won't focus on the AutoMapper setup, it's just adding the profile to the configuration and loading that into a mapper. After that I create a `Person` instance to map to an `Employee`.

To make verification easy, I serialise the `employee` variable to json and then I run it through [ApprovalTests](https://github.com/approvals/ApprovalTests.Net) `VerifyJson` method. In the first run, the ApprovalTests framework will generate a file in the bin/Debug folder named _AutMapperTests.TestMapping.received.json_. It will also fail the test. The name of the file is _<ClassName>.<MethodName>.recieved.json_. By the way, I could also serialise to a number of other formats such as XML or compare pictures and PDFs, but I prefer JSON. The serialised object will be stored in this file. I review the content of this file so I'm sure all the information is correct and the mapping is done as intended.

```
{
  "Id": 1,
  "Name": "Ken Bonny",
  "EmployedSince": "2018-01-01T11:00:00"
}
```

To get the test passing, all I have to do is rename the file from _AutMapperTests.TestMapping.received.json_ to _AutMapperTests.TestMapping.approved.json_. The test framework will recognise the file and use the content to check against the input that is passed to it.

Do make sure all the data is preset or that the function that creates the `Person` instance creates the same information again and again, every time the test is run. For example, if you have a system that creates a person and puts the date of today in the `EmployedSince` field, make sure you have a system that inserts the same date every time. Otherwise the ApprovalTests framework will fail the test because the approved file contains a different date.
