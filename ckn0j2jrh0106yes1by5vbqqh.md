---
title: "Unit testing part 3: Running the tests and calculating code coverage"
datePublished: Tue Sep 13 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j2jrh0106yes1by5vbqqh
slug: unit-testing-part-3
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617381304579/GE2hscjzg.jpeg
tags: csharp, unit-testing, testing, dotnet, dotnetcore

---


_This [series of posts](https://kenbonny.net/tag/better-code-with-tests/) accompanies a talk I give about unit testing with the [xUnit test framework](https://xunit.github.io/). Any reader who saw my talk can use this as a reference, others can use this as a starting point to write even better maintainable and reliable code. In my talk, I highlight patterns that work well with easily testable code and combine it all into a working application. All code from these articles can be found in a [repository](https://github.com/KenBonny/IronMan) on my GitHub account._

Now that the first unit test is written, I want to execute it to see wether it succeeds or fails. This is facilitated by test runners. I will be covering three: the Visual Studio test runner, the console test runner and the [ReSharper](https://www.jetbrains.com/resharper/) test runner.

## The test runners

Each runner will execute the tests and show the result in a clear way. That said, some test runners are more intuitive than others. I think it will be obvious which one has my preference. To run the Visual Studio or console test runner, I will need to install two NuGet packages. The latest version of ReSharper has a test runner for xUnit built in. Search in NuGet for **xunit runner** and all possible xUnit test runners will show up. I install _xunit.runner.visualstudio_ and _xunit.runner.console_.

## A succesfull test

First up is the Visual Studio test runner. I open the test runner via **Test > Windows > Test Explorer** menu. This will open a tool window that lists all available tests. To discover all tests, I have to build the solution. There is a convenient **Run All** link at the top of the window. The runner shows success by displaying a green icon next to the test name as well as how long the run took. If I click on a test, at the bottom of the window I get details of the last run. If there are errors, they will appear there.

![vs-run-success](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381295296/mt743HqF5.jpeg)

Next up is the console runner. I open a powershell or command window and navigate to the _IronMan\\IronMan.UnitTests\\bin\\Debug_ folder. From there, I execute the following command to run the unit tests. If there is a newer version of the console runner available, update the path accordingly. This will run the tests and print the results in the console.

```
..\..\..\packages\xunit.runner.console.2.1.0\tools\xunit.console.exe .\IronMan.UnitTests.dll
```

![console-run-success](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381296799/z3RHJFQXv.jpeg)

The last one is the ReSharper test runner. I can open the runner by navigating to **ReSharper > Unit Tests > Unit Tests**. ReSharper will discover tests as I write them, so a list of available tests will be shown quite quickly. I run one or more tests by selecting them and clicking the green arrow above the listed tests. The results are displayed in a new window. This is the test session explorer and contains the results of the current run. ReSharper allows me to define different sessions (which can be run and rerun as I like) to test different parts of my application. This allows me to create a session with only the tests of the part of the application I'm working on without needing to run all tests. This is especially convenient when the test base grows. When I click on a test, on the right side I see an overview of the test. With a successful run, there will be very limited information.

![r#-run-success](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381298385/Y16FljHbq.jpeg)

I have the ReSharper Ultimate package which gives me access to profiling, coverage and memory management inspection tools. These are very convenient tools for checking code coverage (more on this later) or measuring other metrics.

## A failed test

Now let's see what happens when I have a failing test. I will change the second assert from True to False and run the test again. I chose the second assert because this one has a comment in it.

In the Visual Studio test runner I get the following result. It shows a red X next to the test to indicate it failed. The result pane shows the source of the failure and the message I have set to clarify what happened and it displays the expected and actual values. Then it displays a limited stack trace so I can diagnose what went wrong. I can click on the blue parts to navigate quickly and easily to the code that causes the problem.

![vs-run-fail](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381299822/oxG4P2TSk.jpeg)

In the console, I get a similar result by running the previous command again. It shows the failed test in red, displays the message and expected and actual values together with the stack trace. No easy clicking here unfortunately.

![console-run-fail](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381301341/z5_o2gxL6.jpeg)

In the ReSharper test runner, I get again a similar result. In my opinion, it is better structured and the stack trace is more detailed, giving a more readable view of where the error occurred. Here are also code references for easy navigation.

![r#-run-fail](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381302807/bIPtG-zZA.jpeg)

So each test runner shows the same information, but in a slightly different way. I do not need to buy external tools such as ReSharper to work with xUnit. The Visual Studio and console runners give me all the information I need to check what is wrong. ReSharper just makes my life much more easy.

## Code coverage

The way I'll write code in the next chapters is a bit different from how traditional TDD works. In contrast to the previous chapter, I'll write code before there are tests. Writing tests first is the best approach. It delivers better software and avoids pitfalls. For demonstration purposes, I have code ready and I will add tests later (during my talk/while going through these articles). Sometimes I write code first and then add tests later. This is less efficient, but more practical (opinions may vary).

Do **NOT** skip writing tests. This is easy to do because the code looks good. Looks can be deceiving. Be sure. One way of being sure is to check [code coverage](https://msdn.microsoft.com/en-us/library/dd537628.aspx). This is a metric, mostly expressed in percent, that shows how much of my actual code is being run through one or more tests. I like to set 80% coverage as a goal. If I don't have the time or resources to go that high, then I try to test the most important decision logic of my application. Code coverage doesn't indicate how good my tests are, it just shows how much of the application is being tested. Focus on correct behavior first.

ReSharper has an addon, [dotCover](https://www.jetbrains.com/dotcover/), that lets me inspect how much of the code is covered and also highlights the covered statements. I could do the same in Visual Studio via **Test > Analyze Code Coverage > Selected Tests** or **All Tests**.

## Conclusion

All test runners give a nice overview of passing and failing tests with enough information to start fixing bugs.

For the next article, I'll expand regular tests with data driven tests to facilitate test reuse. If anybody notices any inconsistenties, please contact me on [twitter](https://twitter.com/bonny_ken/) or via [email](mailto:bonny.ken@gmail.com).
