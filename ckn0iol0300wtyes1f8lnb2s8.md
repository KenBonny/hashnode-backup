---
title: "A rant about bad code"
datePublished: Mon Aug 27 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iol0300wtyes1f8lnb2s8
slug: a-rant-about-bad-code
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380653888/AAFQxo6QT.jpeg
tags: refactoring, csharp, dotnetcore, solid-principles

---


Last week I stumbled upon code that I could not let go unrefactored. The developer who wrote it already got to hear this rant, but I thought I'd repeat it just so my nice readers won't make the same mistakes.

As is tradition, I can't directly talk about the problem domain because non-disclosure agreements. Aren't they fun. So today, the narrative is a book. In the initial module, there could be only one version of a book. After the change, a book has a draft before it becomes a version. A version cannot be edited. A draft can be created from a previous version which will become a new version in time. There can only be one draft.

What set off alarms in my head was when I saw that a lot of the original functionality was still there. I could still update any version, not just a draft, or I could create multiple drafts. So I investigated further and found more problems in the updated code.

The most important lesson I imprinted on the poor junior though: YOU DO NOT COMMENT OUT TESTS! I can understand ignoring a few tests because they are broken and need to be fixed later when the refactoring is done. This is not good practice, the tests should be fixed while refactoring the code because they tell you what you are breaking. Sometimes this can lead to surprising results, such as tests in the delete functionality that break while I am editing the creation logic.

No matter how useful tests are, they can sometimes be a pain in the ass. But under no circumstances can I condone commenting out the body of a test. To be sure we're all on the same page here, the tests literally  looked like this:

```
[Test]
public void EpxlainsWhatTheTestDoes()
{
  // SetupTestData();
  // var result = ExecuteSystemUnderTest();
  // Assert.AreEqual(expectedValue, result.Value);
}
```

What happens is that the tests show up green, because nothing tells the test that it fails. So the whole test suite shows green while a full test class is commented out. That is just a bad place to be because it erodes trust in the tests. Test should be a source of: everything is green so we should find (close to) no bugs. Instead, I now find myself wondering, how many tests in other parts of the codebase are commented out...

The next lesson is that it's important to use the business language in your code. Don't just talk about a book and versions. Make sure the book has an notion of a draft as well. Make the draft easy to discover. When saving a draft, discard the previous draft so that you can only have one draft. Keep your logic and names in line with what the business needs.

Continue that line of thinking into your tests. Add tests with descriptive names such as `WhenSavingASecondDraftThenTheFirstDraftIsRemoved` and have that test check for that business specific rule. Don't start checking other specifications, write specific tests for those things. That way, you can find out exactly what went wrong, because your tests will tell you.

Talking about separating responsibilities, using the same model to pass to the create and update function as well as returning it from a query method is not advisable. I require different information when creating a draft versus updating a draft and the model contains even more information for when it contains all the information from a query.

```
public class BadVersionModel
{
  public int Id;
  public int BookId;
  public string VersionText;
  public DateTime? PublicationDate;
  public bool IsDraft;
}
```

This creates confusion when consuming the module. I would question why I would need to specify the identifier or publication date when updating a draft. Or why I would need to specify that this is a draft when I know I'm creating a draft. Similar questions would pop up when updating the draft. This lead to a bug as the version identifier could be used to change a published version while updating a draft.

During the refactoring, I created different models for different scenarios that only ask for specific information that is needed. When I create a draft, I want to create one from a previous draft. (The first draft is created when creating a book.) So I would only need the identifier of an existing version to create a draft.

```
public class CreateDraftModel
{
  public int VersionToBaseDraftOn;
}
```

The model also has a very descriptive name, almost a command. That is no coincidence, because it tells the consumer what they are doing. The other models will have more descriptive names as well.

The update model will need just the book identifier as there will only be one draft, so I'll be updating that books draft. It will need the new text for the version so it can overwrite the existing text.

```
public class UpdateDraftModel
{
  public int BookId;
  public string DraftText;
}
```

Again, instead of using the `VersionText`, I name the draft text `DraftText` to convey more meaning. Being generic is harder to understand than being specific, it's sometimes hard to find a balance. I experiment with names until I find one that I think fits what I want to communicate.

The last model for the queries contains all the information that I need to display the versions.

```
public class VersionModel
{
  public int Id;
  public int BookId;
  public string VersionText;
  public DateTime? PublicationDate;
  public bool IsDraft;
}
```

The initial model works very well in the query scenario. So I kept it. The problem was that this model contained too much and too ambiguous information (`Id` vs `VersionToBaseDraftOn`). With the updated models, there's no room for interpretation, nor questions what information is needed in each scenario.

Furthermore, information that I don't need, because I already know it or I can infer it, I don't ask. For example: I know when a version is a draft.

After the changes in the back, there was some cleaning up to do in the front as well. Most of those changes related to the same issues I already tackled. Meaning I changed where the "BadVersionModel" was partially used to the specific models and I updated some naming.

There is one place in the UI I want to focus on. There is a popup when creating a book in which the title is entered. The popup is designed to be generic so it can ask for different text inputs throughout the application by injecting the text to display when it pops up. My junior wanted to keep the popup really generic. That is why he added a switch that can display a date picker when asking for the publication date instead of the text input.

The result is that the popup has a dozen checks when it should display the date time picker, how fields should be validated and what model it should return. I split the date picker component into a different popup and it simplified the code tremendously. Afterwards the code was a lot easier to read and to reason about what was going on. No more remembering that in this case it should use a date picker and in that case it should just be text. Two different popups that each have their own purpose, yet are still reusable within their intended use cases.

Observant readers will have noticed that all errors refer to the [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle) from the [SOLID principles](https://en.wikipedia.org/wiki/SOLID). That is the focus of this post. In code, keep responsibilities separated. It will make the code easier to read and easier to think about because each part only does one thing: either ask for a text or ask for a publication date, but not both.
