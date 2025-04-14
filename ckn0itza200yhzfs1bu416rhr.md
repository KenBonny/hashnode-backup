---
title: "Generating SpecFlow files in Rider"
datePublished: Mon May 28 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0itza200yhzfs1bu416rhr
slug: generating-specflow-files-in-rider
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380905840/lF7tZqC2E.jpeg
tags: ides, dotnet, dotnetcore, cucumber

---


At the moment, there is no official [SpecFlow](http://specflow.org/) support in [Jetbrains .net IDE Rider](https://www.jetbrains.com/rider/). I found a way to make it more bearable. Keep in mind that it's not a perfect solution.

After I install the SpecFlow nuget package, it downloads the SpecFlow executable that generates the _.feature.cs_ files. Combined with the [file watcher plugin](https://plugins.jetbrains.com/plugin/7177-file-watchers), I can let the code behind files be generated. Go to Settings > Tools > File Watchers and click on the green + icon to add a custom file watcher.

Give the watcher an easy to understand name such as "SpecFlow file generation". Select the _File Type_ "Cucumber Scenario". The _Scope_ isn't really important, because the generator will refresh all _.feature.cs_ files. I set the _Scope_ to "Open files". The _Program_ to run is situated in the folder "<solution folder>\\packages\\SpecFlow<version>\\tools\\specflow.exe". The _Arguments_ are "generateall $ProjectFileDir$\\<TestProjectName>\\<TestProjectName>.csproj". The last setting is _Output paths to refresh_ and it should be "$ProjectFileDir$".

```
File Type: Cucumber Scenario
Scope: Open Files Program: <solution folder>\packages\SpecFlow<version>\tools\specflow.exe
Arguments: generateall $ProjectFileDir$\<TestProjectName>\<TestProjectName>.csproj
Output paths to refresh: $ProjectFileDir$
```

![1-config](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380904164/ylGfrGNsc.png)

The "generateall" argument needs the test project _.csproj_ file to see which [cucumber](https://cucumber.io/) files it needs to inspect to generated the _.feature.cs_ files. The _Output paths to refresh_ tells rider which folder to inspect after the program has run to detect which files are changed.

The biggest remaining problem for me is that there is no automatic steps generation. I usually start Visual Studio to generate the steps file the first time and then I manually edit the file whenever I update or add a step.

In my team, we use a Visual Studio tool to nest the _.feature.cs_ and _.steps.cs_ files. In Rider, there is an option to [nest files](https://www.jetbrains.com/help/rider/File_Nesting_Dialog.html). At the top right corner of the Solutions window, click the gear icon and select the _File Nesting._.. menu item. Then click the + icon to add a new rule and add rules for both the _feature.cs_ and _.steps.cs_ files. It won't nest the files in the solution, it will just display them as nested. So Visual Studio won't nest the files, be aware of this.

It's not perfect and official support would be better, but it's a start. Now lets hope either Jetbrains or Specflow releases an official plugin for Rider so we can all enjoy SpecFlow as easily as it is in Visual Studio. If you want it to get here faster, start bothering both [Jetbrains](https://rider-support.jetbrains.com/hc/en-us/community/posts/207696605-Specflow-Add-on) and [SpecFlow](https://github.com/techtalk/SpecFlow/issues/773) about this.
