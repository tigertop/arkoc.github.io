---
layout: post
title: Using MSBuild for DLL configuration files transformations and output to referencing projects
---

#### What we are going to archive?

Consider folowing setup:

1. ProjectA (Class Library) with custom configuration file foo.config (with configuration transformations)
2. ProjectB (WebProject, ConsoleApplication and so on) that references ProjectA and includes the ProjectA's transformed configuration file foo.config (in output folder of ProjectB)

#### Let's start.

### Creating configuration file with transformations for DLL (Class Library)

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
