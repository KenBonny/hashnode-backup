---
title: "Resharper shadow copies assemblies before test runs"
datePublished: Mon Oct 30 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0izniq010czfs15l2iejk8
slug: resharper-shadow-copies-assemblies-before-test-runs
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617381170345/WrC0ERWHB.jpeg
tags: unit-testing, ides, testing

---


While writing an XML parser I customised, my unit tests all failed with an error message along the lines of "Could not load file 'C:\Users[username]\AppData\Local\Temp[guid]\Data\test.xml' or one of its dependencies. The system cannot find the file specified.".

My test solution contains a folder _Data_ in which I put a _test.xml_ file that contains an example of the data I need to parse. This allows me to test both reading and parsing the file in a single test. The problem is that the test can't find the xml file on my laptop and on a colleagues laptop all tests are green.

So after a bit of googling, I found out that [Resharper](https://www.jetbrains.com/resharper/) is doing some [clever shenanigans](https://stackoverflow.com/questions/16231084/resharper-runs-unittest-from-different-location). Resharper copies the assemblies to a test folder in the '~\AppData\Local\Temp[guid]\'. So if the tests run from that location, you won't see errors that the assemblies are in use by another process when you build the solution while tests are running.

To prevent Resharper from shadow copying the assemblies to the temporary location, open the Resharpers options, go to the _Tools > Unit Tests_ page and under the _Unit Test Runner_ chapter, disable the _Shadow copy assemblies being tested_.

In most test scenarios, this won't cause any problems and is a rather useful feature to have. Whenever this isn't convenient, I hope my bumbling around will save somebody else an hour of their time.
