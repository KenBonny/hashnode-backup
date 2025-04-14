---
title: "Deploy automation on a smaller scale"
datePublished: Mon Jan 23 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0isjyv00y3zfs12nuzgx01
slug: deploy-automation-on-a-smaller-scale
tags: deployment, automation, dotnet

---


Automatically deploying software doesn't only help huge projects that have set up their own [Octopus deploy](https://octopus.com/). It can also help on a smaller scale.

To help my wife with her job, I build her a [CLI tool](https://github.com/KenBonny/KellyHelper) to simplify some tasks (i.e. calculate days between two dates). To simplify deploying the software, I automated this part of the project. My requirement is that each time I build in _Release_ mode, the executable and all dependant binaries get automatically zipped and placed in a shared folder. I use [Dropbox](https://www.dropbox.com/) for this, but any sharing service will work equally well.

Post-build events come in very handy: set up once, never get it wrong again. The first requirement is that it will only execute when I build in _Release_. I don't want all the _Debug_ builds to update the shared folder, which could put broken or buggy software in the shared folder. I don't want my automated builds to put bad software out there.

First I open the project properties of the project with the executable, I open the _Build Events_ tab and in the _Post-build event command line_ text box I type:

```
if $(ConfigurationName) == Release (
)
```

The `$(ConfigurationName)` contains the build variable so I can verify I'm in the correct build. There is a detailed list of [environment variables](https://msdn.microsoft.com/en-us/library/c02as0cs.aspx) on MSDN.

The next step is to zip the necessary files. My favourite zipping utility is [7zip](http://www.7-zip.org/), which is easily accessible via the command line. I add the following line between the brackets:


```
"C:\Program Files\7-Zip\7z.exe" a -tzip "[dropbox-folder]\$(SolutionName).zip" "$(TargetPath)" "$(ProjectDir)$(OutDir)*.dll"
```

That's a mouthful, so lets dissect it. The `"C:\Program Files\7-Zip\7z.exe"` is the location of 7zip, so it can run the zipping tool.

`a -tzip` indicates that the files should be added to an archive and the zip protocol should be used. 7zip also supports a number of other zipping formats, but I'm not sure that my wife's work PC can unpack those formats. Zip is supported by Windows out of the box.

The next part is the location and name of the zip archive: `"[dropbox-folder]\$(SolutionName).zip"`. The `$(SolutionName)` is the name of the project that is being build. If I ever decide to change the name of the project, then I don't need to update the build event.

The last parts are the items I want to add to the archive: `"$(TargetPath)" "$(ProjectDir)$(OutDir)*.dll"`. The `$(TargetPath)` is the executable of the project, in another project it could be a .dll. This ensures I add the executable to the archive. I experimented with a little architecture in the project so I put the processing in a separate project and used [Autofac](https://autofac.org/) to wire it all together. Overkill, yes, but it was fun. It also means that I have extra .dlls I have to include in the archive. I do this by specifying `$(ProjectDir)$(OutDir)*.dll`. The two environment variables form the full path to the output directory. The `*.dll` is a filtering function of 7zip that selects all .dll files.

The whole post-build event looks like:

```
if $(ConfigurationName) == Release (
  "C:\Program Files\7-Zip\7z.exe" a -tzip "[dropbox-folder]\$(SolutionName).zip" "$(TargetPath)" "$(ProjectDir)$(OutDir)*.dll"
)
```

### Add assembly version to archive name

Now I could stop here, because 7zip would add and overwrite files in the archive, but I do have another feature I want to add: versioning. I want the archive name to include the assembly version of the executable to keep multiple versions. So even if I screw up, there is a workable previous version available.

Unfortunately, out of the box there is no variable or macro available that gives the assembly version. After a bit of googling, I came across [this Stack Overflow answer](http://stackoverflow.com/a/19371257) that is, in my opinion, the easiest solution.

To begin, close Visual Studio and navigate in the Windows Explorer to the project folder. There is a file named "\[ProjectName\].csproj" (or .vbproj for some others). Edit this file in a non Visual Studio editor (VS will open the project, not the file) such as Visual Studio Code, Notepad, Notepad++ or any other editor. At the bottom, add following XML:

```
<Target Name="PostBuildMacros">
  <GetAssemblyIdentity AssemblyFiles="$(TargetPath)">
    <Output TaskParameter="Assemblies" ItemName="Targets" />
  </GetAssemblyIdentity&amp;amp;gt;
  <ItemGroup>
    <AssemblyVersion Include="@(Targets->'%(Version)')" />
  </ItemGroup&amp;amp;gt;
</Target>
```

Then look for the location where your macro is defined, it will look like:

```
<PropertyGroup>
  <PostBuildEvent>if $(ConfigurationName) == Release (
"C:\Program Files\7-Zip\7z.exe" a -tzip "[dropbox-folder]\$(SolutionName).zip" "$(TargetPath)" "$(ProjectDir)$(OutDir)*.dll"
)</PostBuildEvent>
</PropertyGroup>
```

Modify it so it looks like the following xml:

```
<PropertyGroup>
  <PostBuildEventDependsOn>
    $(PostBuildEventDependsOn);
    PostBuildMacros;
  </PostBuildEventDependsOn>
  <PostBuildEvent>if $(ConfigurationName) == Release (
"C:\Program Files\7-Zip\7z.exe" a -tzip "[dropbox-folder]\$(SolutionName)-@(AssemblyVersion).zip" "$(TargetPath)" "$(ProjectDir)$(OutDir)*.dll"
)</PostBuildEvent>
</PropertyGroup>
```

The biggest change is that I added the whole `PostBuildEventDependsOn` tag. This references the custom `Target` I build earlier. There is also a second, more subtle change. I added `-@(AssemblyVersion)` to the archive name. This is the macro defined in the custom `Target` from earlier. Also note that I use an `@` to indicate the macro and not a `$` which references a variable.

With this in place it adds the assembly version nicely to the name of the archive. This concludes how I handle automatic deployment in a small project. Now to find a way to automatically update the assembly version when I build in Release. That's for a next post.
