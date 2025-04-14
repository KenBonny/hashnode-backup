---
title: "String manipulation kata implementation"
datePublished: Mon Jun 26 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j0x5p00zzyes110f0ak92
slug: string-manipulation-kata-implementation
tags: csharp, programming, dotnet, dotnetcore

---


Since I wrote a [simple kata](http://kenbonny.net/2017/06/19/string-manipulation-kata/) to run an experiment, I implemented a solution. Lets look at the outcome.

A simple experiment with, what I think, is an elegant solution. I won't go into the details of all the capitalisation code, I think anybody could invent something like that. This kata revolves around the infrastructure, how the requests are routed to the right capitalisation code. Just know that all capitalisation code is behind an interface `ICapitaliser` with one method `string Capitalise(Command command);`.

The first thing I did was import the [CommandLine](https://github.com/gsscoder/commandline) package, which is basically a mediator. Instead of naming it options, I called it commands. The base command is very simple:

```
public abstract class Command
{
[Option('t', "text", Required = true, HelpText = "The text to capitalize.")]
public string Text { get; set; }
}
```

So the text to capitalise will be in every command. Then I have an array of specific commands for every capitalisation option.

```
[Verb("lower", HelpText = "All letters to lowercase.")]
public class AllLowerCaseCommand : Command { }
```

This is the whole specific command. Simple experiment, simple and elegant commands.

The easiest method would be to process the command in the `MapResult` function of the CommandLine package. I choose to initialise the `ICapitaliser` in the `MapResult` function and return the command.

Originally, the initialisation of the different `ICapitaliser` was a bunch of if statements after another. I just updated this to use a `Dictionary&lt;Type, ICapitaliser&gt;` where the `Type` is the type of the `Command` and the `ICapitaliser` is an instance that will properly process the corresponding command.

Are there other ways to do this: yes, but I like the way I solved it. If you solve this kata in another way, do let me know. My contact details are at the top of the page or check out my [contact page](http://kenbonny.net/about/).

Edit: apparently I forgot [the code](https://github.com/KenBonny/KenBonny.StringManipulationKata).
