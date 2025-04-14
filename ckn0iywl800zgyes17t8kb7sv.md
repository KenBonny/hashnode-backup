---
title: "Partially applying TDD doesn't work"
datePublished: Mon Jun 05 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iywl800zgyes17t8kb7sv
slug: partially-applying-tdd-doesnt-work
tags: unit-testing, testing

---


In my current job, I've heard dismissive talk about testing. Along the lines of "well, that's cute that you did that, now get back to work". Work being manual tests to make sure everything works as intended.

For now, they are right. The problem is that there are too few automated tests. Most of the code base is not tested. This means that even when the tests are run, the information is largely useless as it only gives an image of a small portion of the code. Because there are few tests, when the code is actually executed in a test or production environment, there are unwanted effects (such as nulls or unexpected values that happen only in edge cases that weren't considered).

The lack of tests causes everybody to be afraid when changing code. Few know what will actually happen if something is changed. This situation is increasingly amplified because only a few people on the team write tests. Each day, quite a bit of code is added without tests, so the amount of untested code increases daily. This leads to more uncertainty, manual testing and unwillingness to change existing code.

The current system is very tightly coupled. This means that it's very hard to test. If I want to test some functionality, most of the time I have to initialise a connection to the database and instantiate other modules. I cannot find an easy way to work around them without having to either refactor half the code base or spend hours on setting up complex mocks.

This is the result of writing the tests after the code, in this case a decade later. Because in the present it's easy to tightly couple the system. This comes from a "never mind, we'll worry about that when the time comes"-mentality. Well, the time has come and changing the system takes a lot of time. This includes the time it takes to customise the system for clients which is hurting the overall company.

The solution is simple, but takes a lot of discipline and time. For every line that needs to be modified, tests should be added. The tests that should have been written when the code was being produced will have to be added one case at a time. This will be a painful, time-consuming process which will require every member of the team. If somebody doesn't write tests for the scenario's, new holes in the test coverage will appear and old ones won't be closed. This will only work if everybody is on the same team: team unit test.

All new code will need to be unit and integration tested, one bit at a time. This time with tests up front, not afterwards. Tests up front mean that the code that is produced is more decoupled and easily testable. It also means that external modules can easily be replaced because fakes and mocks should easily take the place of those real modules.

More loosely coupled code means that code can more easily be refactored or replaced without affecting other code. This leads to a more stable system where everybody wins. Having a reliable test suite means that developers can be more confident when refactoring existing code. A lot of refactoring, leads to a lot of experimentation in which the best outcome can be found. Another win for the developers, the company and ultimately the customers.
