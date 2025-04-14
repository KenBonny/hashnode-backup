---
title: "Tests ran multiple times on the CI server"
datePublished: Wed Dec 07 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j1c4y010xzfs158ly3ufl
slug: tests-ran-multiple-times-on-the-ci-server
tags: programming, continuous-integration, testing, general

---


A while ago, colleagues and I encountered strange behaviour on our Continuous Integration server: several tests were run twice.

The problem was caused by a test project that was referenced in another test project because of some infrastructure code that was reused. Probably a [Visual Studio](https://www.visualstudio.com/) or [ReSharper](https://www.jetbrains.com/resharper/) suggestion of "Would you like to reference ClassX from Project.Y" and somebody just clicked "accept" or maybe even ReSharpers [auto resolve all references of pasted code](https://www.jetbrains.com/help/resharper/2016.2/Coding_Assistance__Importing_Namespaces.html). This caused a test assembly to be loaded twice and thus the tests were discovered twice. Assemblies should not be carelessly referenced.

We solved it by putting all general purpose infrastructure code into a separate assembly that can be safely referenced throughout all the test projects. Case closed.
