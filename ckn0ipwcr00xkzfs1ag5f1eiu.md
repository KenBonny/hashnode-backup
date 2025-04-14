---
title: "Checking aliases in Have I Been Pwned"
datePublished: Mon Apr 16 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0ipwcr00xkzfs1ag5f1eiu
slug: checking-aliases-in-have-i-been-pwned
tags: csharp, security, dotnetcore

---


I like [Have I Been Pwned](https://haveibeenpwned.com/). I love the simplicity. Unfortunately, it lacks support for Gmail plus notation.

The plus notation, also known as [an alias](https://support.google.com/mail/answer/22370?hl=en#alias), is very convenient to know how my email address is used by the sites I provide it to. When I sign up for a service, I put the domain name after the plus. Example: example+linkedin@gmail.com. Now I know how many services buy my email from LinkedIn based on the spam I receive that does not come from a LinkedIn address. It's more than I like.

Have I Been Pwned does not support it because it would add quite the overhead to filter out the + and everything after it. I asked [Troy Hunt](https://www.troyhunt.com/) about it and apparently, only about 0.03% of the email addresses in his database uses the alias notation. So I agree that it's not worth it for him to support Gmail aliases.

%[https://twitter.com/troyhunt/status/692328070506749952]

So I created a command line tool that checks emails against the [Have I Been Pwned API](https://haveibeenpwned.com/API/v2). The code of this tool can be found on [a repository](https://github.com/KenBonny/KenBonny.CheckHibp) on [my GitHub profile](https://github.com/KenBonny/). I have compiled versions for [Windows 10 (x64)](https://github.com/KenBonny/KenBonny.CheckHibp/blob/master/win10-x64.7z) and [OSX](https://github.com/KenBonny/KenBonny.CheckHibp/blob/master/osx.7z).

Small note on the OSX version, I do not own any Apple products, so I have not been able to test the OSX deployment. If there are bugs, [get in touch](http://kenbonny.net/contact/) with me. Also, for OSX, the [dotnet core runtime](https://github.com/dotnet/docs/blob/master/docs/core/tutorials/using-on-macos.md) should be installed.

How does it work?

1. Create a text file with an email address on each line.
2. Run the following command.

On Windows 10:

```
.\KenBonny.CheckHibp.exe [location of email file]
```

On OSX:

```
dotnet run KenBonny.CheckHibp.dll [location of email file]
```

That's it, just let the program spit out which email address is pwned in which breaches. An email that isn't found will produce a line like:

> 0001 : Check some.email@gmail.com => Not Found

An email that is found in Have I Been Pwned will produce a line like:

> 0001 : Check test@gmail.com => LinkedIn

The list of emails are from my password manager: [1Password](https://1password.com/). I wrote a small tool to get all the email addresses out (I apparently have over 100 different email addresses). I'm not going to publish that tool, because it runs over the raw export of my 1Password vaults. Which would mean the tool also has access to all passwords, one-time passwords and all secrets in the vaults I supply it. I do not want the responsibility of providing a tool that anybody just "has to trust" with all their private data. So getting the list of email addresses is your responsibility.

Have fun checking a list of email addresses. If the application seems to check addresses slowly, that's because I wait 2 seconds between each check to stay out of the [rate limit](https://www.troyhunt.com/content-images-2016-09-a-one-week-traffic-snapshot-1-png/) on the Have I Been Pwned API.
