## Using SpecFlow 3.0 with Rider

> Checkout the [Rider plugin for SpecFlow](https://plugins.jetbrains.com/plugin/15957-specflow-for-rider) for an easier integration!

A while ago, I [wrote](https://kenbonny.net/2018/05/28/generating-specflow-files-in-rider/) about using [SpecFlow](https://specflow.org/) with [JetBrains IDE Rider](https://www.jetbrains.com/rider). Recently, SpecFlow updated their version to 3.0 and it brings some different behaviour with it. After using it for a while, I really like the new flow.

The first change is that I need to install three packages, instead of two. [SpecFlow](https://www.nuget.org/packages/SpecFlow) and a unit testing framework (MSTest, [NUnit](https://www.nuget.org/packages/NUnit/) or [xUnit](https://www.nuget.org/packages/xunit/)) still need to be installed. Additionally, [SpecFlow.Tools.MsBuild.Generation](https://www.nuget.org/packages/SpecFlow.Tools.MsBuild.Generation) needs to be installed as well.

The biggest benefit of the SpecFlow.Tools.MsBuild.Generation nuget is that I don't need to set up a file writer anymore. I do need to update the .csproj file to include some configuration. The whole process is [nicely documented](https://specflow.org/documentation/Generate-Tests-from-MsBuild/) on the SpecFlow site, but I'll give the basics here.

![Edit csproj file](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381378574/nMNKrnVFq.jpeg)

First, right click on the test project and choose _Edit_ > _Edit MyProject.csproj_ option. Then just paste in the next bit of XML:

<Target Name="AfterUpdateFeatureFilesInProject">
    <!-- include any generated SpecFlow files in the compilation of the project if not included yet -->
    <ItemGroup>
        <Compile Include="\*\*\\\*.feature.cs" Exclude="@(Compile)" />
    </ItemGroup>
</Target>

The old SpecFlow version can add configuration tags in the _.csproj_ file: `` `<Generator>SpecFlowSingleFileGenerator</Generator>` ``. They can easily be replaced by doing a search and replace (ctrl+h in rider) and using this regex to find all of them: `` `\n[ ]+SpecFlowSingleFileGenerator` ``.

Save the .csproj file and just build the project. The _.feature.cs_ files will be generated next to the _.feature_ files. You can include them in the project, but the build server will update the generated files when it builds the project anyway. So you don't need to include the _.feature.cs_ files in the project anymore if you don't want to.

Happy coding and keep those tests green!

_Update_: I got some feedback that it isn't intuitive to find the _Given/When/Then_ declarations for the _.feature.cs_ file. All I need to do is run the tests. The method definitions appear in the test output window. If I add a new line, I just rerun the test. The test will be marked incomplete with the new method signature in the test output window. All that's left to do is to copy and paste the signatures into the right file and flesh out the new method. Oh, and don't forget to add `Binding` and `Scope` attribute on the class. I've forgotten that more times than I dare to admit.
