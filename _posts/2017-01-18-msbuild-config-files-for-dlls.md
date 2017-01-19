---
layout: post
title: Using MSBuild for DLL configuration files transformations and output to referencing projects
---

#### What we are going to archive?

1. ProjectA (Class Library) with custom configuration file foo.config (with configuration transformations)
2. ProjectB (WebProject, ConsoleApplication and so on) that references ProjectA and includes the ProjectA's transformed configuration file foo.config (in output folder of ProjectB)

<!--more-->

#### Let's start.

Side Note: Transformations are nessaccary to have different settings for different build configurations. (e.g. Release, Debug, Staging)

Let's start by adding a configuration file to our Class Library project. Right Click on Project > Add > New Item > Application Configuration File. Name it foo.config.

Unload the project by clicking "Unload Project" from project's context menu. Right click on unloaded project and click "Edit .csproj file".

Add/Modify this section before (`<Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />`)


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
 
 
Now we need to `MSBuild` task to transform our `foo.config` based on configuration. Because of we need include our configuration file as `Content` and copy it to output directory (to referenced project) we need to transform our file before build starts.

Add following section below (`<UsingTask TaskName="TransformXml" AssemblyFile="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\Web\Microsoft.Web.Publishing.Tasks.dll" />`)


```xml
  <Target Name="BeforeBuild">
    <MakeDir Directories="config"/>
    <TransformXml Source="foo.config" Destination="config\foo.config" Transform="foo.$(Configuration).config" />
  </Target>
```


Now last step. We need to make our transformed file to be included in output directory (also in directory where our dll is referenced)

Add this section after (`<Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />`)


```xml
  <ItemGroup>
    <Content Include="config\foo.config">
      <Link>foo.config</Link>
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </Content>
  </ItemGroup>
```


Here we go. Our setup is config. Now our dll and configuration file will be copied to ouput directory whereever it is referenced.
