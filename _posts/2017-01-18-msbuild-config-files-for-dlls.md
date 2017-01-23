---
layout: post
title: Using MSBuild for DLL configuration files, transformations and output to referencing projects.
---

#### What we are going to archive?

1. ProjectA (Class Library) with custom configuration file foo.config (with configuration transformations)
2. ProjectB (WebProject or ConsoleApplication, ...) that references ProjectA and includes the ProjectA's transformed configuration file foo.config (in output folder of ProjectB)

<!--more-->

#### Let's start.

Side Note: Transformations are nessaccary to have different settings for different build configurations. (e.g. Release, Debug, Staging)

Let's start by adding a configuration file to our Class Library project. Right Click on Project > Add > New Item > Application Configuration File. Let's name it foo.config.

Unload the project by clicking "Unload Project" from project's context menu. Then right click on unloaded project and click "Edit .csproj file".

Add or Modify the below section just before (`<Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />`)
This will link each configuration's config file to the root foo.config. As a result the files will be displayed as children to the root configuration file in the Solution explorer of Visual Studio, and will inherit settings from root configuration file.

```xml
<ItemGroup>
  <None Include="foo.config">
  </None>
  <None Include="foo.Debug.config">
    <DependentUpon>foo.config</DependentUpon>
  </None>
  <None Include="foo.Release.config">
    <DependentUpon>foo.config</DependentUpon>
  </None>
  <None Include="foo.Staging.config">
    <DependentUpon>foo.config</DependentUpon>
  </None>
</ItemGroup>
```
 
 
Now we need to create a `MSBuild` task in-order to transform our `foo.config` based on selected configuration. As we need to set the root config. file's build action to `Content` and copy it to the output directory (also in referencing project) we need to transform our file before build starts.

Add following section below this (`<UsingTask TaskName="TransformXml" AssemblyFile="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\Web\Microsoft.Web.Publishing.Tasks.dll" />`)


```xml
<Target Name="BeforeBuild">
  <MakeDir Directories="config"/>
  <TransformXml Source="foo.config" Destination="config\foo.config" Transform="foo.$(Configuration).config" />
</Target>
```

Now last step. We need to make our transformed file to be included in the output directory (also in output directories of other projects  where our dll is referenced).

Add this section just after (`<Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />`)

```xml
<ItemGroup>
  <Content Include="config\foo.config">
    <Link>foo.config</Link>
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
  </Content>
</ItemGroup>
```

We are all done. Now our dll and the transformed configuration file will be copied to ouput directory whereever the dll is referenced.
