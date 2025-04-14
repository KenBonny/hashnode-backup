---
title: "String manipulation kata"
datePublished: Mon Jun 19 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j102z010rzfs1aonf0v2v
slug: string-manipulation-kata
tags: csharp, programming, dotnet, dotnetcore

---


A [Code kata](https://en.wikipedia.org/wiki/Kata_(programming) is a great way to experiment with code, but I don't nearly do them as much as I should. I spend a lot of time reading about techniques and frameworks, but I could use more practice.

I've been looking for an easy to understand kata in which I could experiment with several patterns. When I couldn't find one to my liking, I started thinking about one. Maybe I've been looking in the wrong place for katas. If anybody has a great place for code katas, be sure to get in [contact](http://kenbonny.net/about/). If this kata resembles another, I did not know about it, this is something I thought up.

Finally, I thought of a simple to understand kata that can be implemented in several ways.

> Write a program that takes text and a type of capitalisation to apply to the text. Then it should display the text according to the chosen capitalisation.

> Example: the text entered into the application is "example text". Based on the type of capitalisation it should be printed out differently. CamelCase should print "Example Text". AllUpper should print "EXAMPLE TEXT". AllLower should print "example text". Alternating should print "ExAmPlE tExT". DifferentAlternating should print "eXaMpLe TeXt". AlternatingWord should print "EXAMPLE text". DifferentAlternatingWord should print "example TEXT".

As you can see, you can add additional formatters if you'd like. There are also different ways of solving this.

But wait, there's more. Just like a "real application", there are extra features being requested. Only advance to this second part, after completing the first part. This is to challenge you to think about making a modular and extendable program. Evil, I know.

> Once the capitalisation is in place, there is a need to filter out naughty words. This can be done in several ways. One way is to remove the word altogether. Another is to replace the naughty word with a random string of characters.

> Example: The entered text is "fuck this shit". Below I will display the two solutions using the AllUpper capitalisation. The FilterNaughtyWordsOut option should print "THIS". The ReplaceNaughtyWordsWithGibberish option should print " @#%& THIS %@!\*".

This is just an example, if you want to solve this differently, feel free to use your imagination. Remember, this is a fun exercise, if you don't like this, find one you do like. Happy coding!
