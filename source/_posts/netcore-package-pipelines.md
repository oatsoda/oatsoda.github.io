---
title: .NET package versioning and continuous deployment
date: 2020-10-19 13:31:00
tags:
  - Azure DevOps
  - NuGet Packages
  - Pipelines
---

**_Understanding net core latest versioning and packaging_**

I've recently been playing with publishing my first NuGet package into NuGet.org. I always assumed I would eventually need to become familiar with this - specifically using private package management for internal project repositories (for example using [Azure DevOps private repositories](https://docs.microsoft.com/en-us/azure/devops/artifacts/get-started-nuget?view=azure-devops)) but in the end I have gone straight for public NuGet for my first attempt. And it seems my late arrival has worked out well as things are _now_ fairly straight forward!

![NuGet Packages](cartons.png "Source: digital designer https://pixabay.com/users/dapple-designers-7874104/")

## A basic introduction

On my travels I had come across .nuspec files within repositories, so I assumed that this was where I would need to start. However, it seems that since .NET Core, Package management is now a first class citizen, so you can now work directly with your project files.

As you can see, the [.nuspec files are designed for older projects](https://docs.microsoft.com/en-us/nuget/reference/nuspec#project-type-compatibility) which use packages.config files. Now, though, you can just use the [MSBuild reference for packaging](https://docs.microsoft.com/en-us/nuget/reference/msbuild-targets#pack-target).

More on this a bit later.

## A brief history of .NET versioning

If you have used .NET for any length of time you're probably aware of how versioning assemblies can be a bit painful when it comes to adding it to a CI/CD pipeline. Not least because of the whole `FileVersion` vs `AssemblyVersion`! This does of course depend on your level of OCD - but personally I prefer to make sure they are both set the same to avoid any confusion.

Historically with .NET Framework you had to set it in the `AssemblyInfo.cs` file:

```csharp
[assembly: AssemblyVersion("1.0.0.0")]
[assembly: AssemblyFileVersion("1.0.0.0")]
```

If you wanted to automate this on a pipeline you had to use a third party tool or PowerShell script to edit the source code in the pipeline to set the correct version before compilation took place.

Skip forward to netcore and the new project files and things became slightly easier with the `AssemblyInfo.cs` version being replaced with the version inside the project file:

```xml
<PropertyGroup>
   <AssemblyVersion>1.0.0.0</AssemblyVersion>
   <FileVersion>1.0.0.0</FileVersion>
</PropertyGroup>
```

You could then apply a pipeline script or third party tool to set the version in the project file before compilation. Certainly this is how I was doing it from .NET Core 1.0 up to 2.1.

Somewhere along the line though, unbeknown to me, a `<Version>` tag was created which when set alone (instead of separate `AssemblyVersion` and `FileVersion`) ensures that both get set to the given value. Additionally (again, possibly something I missed or which was added at a later date) it is actually possible to set these values from a dotnet command using the `/p` switch without requiring the xml element to even exist within the csproj:

```ps
dotnet build /p:Version=1.2.3.4
```

Which makes the whole CI/CD pipeline very simple to implement and ensure your assemblies are clearly and consistently versioned.

For a full rundown of the various `.csproj` version elements, I recommend [this blog post](https://andrewlock.net/version-vs-versionsuffix-vs-packageversion-what-do-they-all-mean/), but see my note later on regarding the Version Suffix.

## Checking your assemblies

If you want to be sure what version your assemblies have, then opening file Properties will show the File Version, but not the Assembly Version.

Instead, you can use the following PowerShell script to display both versions:

<script src="https://gist.github.com/oatsoda/5f1e7f7e5388810a78905f45007797b9.js"></script>

## Creating your package

Now we know how to set the version and where to set the Package details, we can bring it all together.

Simply add the relevant Package details to your `.csproj` file, for example:

```xml
<PropertyGroup>
  <TargetFramework>netstandard2.1</TargetFramework>
  <PackageId>MyPackageName</PackageId>
  <Authors>My Name</Authors>
  <PackageTags>Single;Word;Tags;Which;Appear;In;NuGet;And;Are;Searchable</PackageTags>
  <Description>A description of your package</Description>
  <PackageLicenseExpression>MIT</PackageLicenseExpression>
  <PackageProjectUrl>https://github.com/me/mypackage</PackageProjectUrl>
  <RepositoryUrl>https://github.com/me/mypackage</RepositoryUrl>
  <RepositoryType>git</RepositoryType>
  <PackageIcon>icon-file.png</PackageIcon>
</PropertyGroup>

<!-- ... -->

<ItemGroup>
  <None Include="icon-file.png" Pack="true" PackagePath="\"/>
</ItemGroup>
```

And you can package it with the `dotnet pack` command:

```ps
dotnet pack /p:Version=1.2.3.4
```

## Adding it to an Azure DevOps Pipeline

Lastly, I wanted to get this implemented in a CD pipeline to make things faster and automated.

If you have used DevOps pipelines before, you'll be aware that there is already a [dotnet pipeline Task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/build/dotnet-core-cli?view=azure-devops) that you can use, for example, running a dotnet build:

```yml
- task: DotNetCoreCLI@2
  displayName: "Build  All"
  inputs:
    command: "build"
    projects: "MySolution.sln"
    arguments: "-c  $(buildConfiguration)  /p:Version=$(Build.BuildNumber)"
```

You can of course change the command to `pack` and this will issue the dotnet pack command as above.

However, I had some issues around the versioning with when it came to pre-release suffixes.

## Pre-release Suffixes

Just when I thought I was done, I realised that I would need to be able to additionally support the suffixes such as `-alpha` or `-beta` so that I can push out pre-release versions of my package. This is simply the case of adding `-alpha` or `-beta` which will be picked up automatically by NuGet.org to mark that version of the package as pre-release.

First of all, I was pleased to find that using the `/p:Version=1.2.3.4-alpha` works just fine - I initially assumed that it would complain about the non-numerics, but it seems it's clever enough to strip this off, setting the `FileVersion` and `AssemblyVersion` to the numerical version, whilst still setting the `ProductVersion` to the entire `1.2.3.4-alpha`.

However, when it came to using the DotNetCoreCLI task I had issues with the versioningScheme parameter. If I set it to `off` and instead use arguments of `/p:Version=` then the package file was not getting named correctly. e.g. it would be called `MyPackage.1.0.0-alpha.nupkg` even if inside (you can open it as a zip file) the `\lib\MyPackage.dll` had the correct File and Assembly version.  After playing with the versioningScheme parameter, I gave up and reverted to just using the dotnet command directly as a PowerShell Task:

```yml
- task: PowerShell@2
  displayName: Package NuGet
  inputs:
    targetType: "inline"
    script: dotnet pack MyPackage.csproj /p:Version=1.2.3.4-alpha
```

## Deploy to NuGet.org

To complete the pipeline, I wanted it to automatically push the package to NuGet.org. My first port of call with this was to investigate the Azure DevOps pipeline NuGet tasks which have various options and configurations for pushing to different NuGet feeds. I had a lot of problems in trying to get these to work. After some digging, it turns out that these NuGet tasks are specifically for pushing to private NuGet feeds, not NuGet.org itself!

In which case I reverted to PowerShell again to call the dotnet nuget push command:

```yml
- task: PowerShell@2
  displayName: "Deploy  to  NuGet.org"
  inputs:
    targetType: "inline"
    script: 'dotnet  nuget  push  ''$(Build.ArtifactStagingDirectory)\*.nupkg''  --api-key  $(nugetApiKey)  --source  ''https://api.nuget.org/v3/index.json'''
```

The API Key is one that you can create when logged in to NuGet.org (using a microsoft account). On NuGet.org, click your username > [API Keys](https://www.nuget.org/account/apikeys) and create a new key. I added mine as a secret variable from within the Azure DevOps portal. There's no need to define this variable anywhere, just make sure that the variable is named the same as the "nugetApiKey" that is used within the YAML.

If you would like to see all of the above in use, [see my Azure DevOps pipeline YAML file](https://github.com/oatsoda/TeePee/blob/main/Release.TeePee.yml).

I may revisit and see if I can get the DotNetCoreCLI task working again with the versioning, but given it is just a wrapper for the `dotnet pack` command, it's not the end of the world.
