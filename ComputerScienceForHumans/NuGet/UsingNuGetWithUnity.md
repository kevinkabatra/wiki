# Using NuGet with Unity
[Unity](https://en.wikipedia.org/wiki/Unity_(game_engine)) is a cross platform game engine that I and so many others leverage as their engine of choice. One drawback that I noticed in Unity is its lack of native support for [NuGet](https://www.nuget.org/) packages, NuGet if you are unaware is a package manager that makes sharing and depending upon code libraries a breeze. I have not researched this but I would assume the lack of support is in part due to Unity's support for various platforms, perhaps building for Android or PlayStation would fail or be too complex. Regardless of their reasoning I still would like to depend on NuGet for a few reasons:
1. Updating a particular dependency is so easy that it is trivial to do, this is important when you are managing multiple assemblies and need to keep the dependencies in sync. Many times in my professional life I have been bit by a dependency on a particular version of Newtonsoft.Json. In our Enterprise Resource Planning (ERP) environment we would then need to search out what version the platform was currently using to ensure that we were in compliance. If we were not we would have to update the path for the reference in each of our projects, where if we had NuGet we could just select the appropriate version from a drop-down list.
1. NuGet is cleaner as it stores all of your downloaded dependencies in one folder regardless of which assembly is using it. This helps eliminate duplicate files which makes maintenance easier. It also helps prevent your project from being dragged into DLL hell. In our ERP environment at work we also need to look in several file locations to ensure that the same DLL has been deployed to N number of locations.
1. Perhaps the biggest benefit is this last one. It manages your dependencies and their dependencies for you automatically. So if I have a dependency on Newtonsoft.Json NuGet will download the dependencies that Newtonsoft.Json has taken based on the framework that I am targeting. This will help prevent compilation and sneaky run-time errors from appearing because a required assembly was not present. For example, if I targeted .NETStandard 1.0 I would need to bring in four additional assemblies in order to properly reference Newtonsoft.Json. Which assemblies do I need? Well without NuGet I could perhaps search the internet in order to locate this answer, or I could just squash the compilation errors as they appeared and hope that I wrote enough unit tests to cover my usage. But I could just use NuGet and click install | update and let the NuGet gods handle the rest of the heavy lifting.

## Without native support, how do you use it?
There have been other articles written about how NuGet can be used. In a previous article I briefly mentioned how I tried to use [NuGetForUnity](https://github.com/GlitchEnzo/NuGetForUnity) but it was not able to properly install [Moq](https://www.nuget.org/packages/Moq/) using its NuGet package. Instead I followed advice from others, use a separate Visual Studio project that can fully utilize NuGet and then manually copy the required assemblies into the Unity Assets folder. Manually? Holy cow, what has been happening in the game development world? Well I tried this approach for my last project, but I only depended upon a single assembly, [Stateless](https://www.nuget.org/packages/Stateless/). Anyone can handle manually managing one assembly.

Recently, due to my love of NuGet, I have decided to start making my own NuGet packages. In my first packages I have decided to create libraries for what I consider common elements to game development. The first is my state machine that controls the [Game State](https://www.nuget.org/packages/Kabatra.Game.StateMachine/), and the second is more work on supporting translation using a [Label Retriever](https://www.nuget.org/packages/Kabatra.Common.LabelRetriever/). The state machine actually depends upon the translation engine, but using NuGet I have no trouble of keeping one updated with the other. The state machine of course also depends upon Stateless, as that is the state machine engine that I used previously. At this point my dependencies will now look like this:
* Label Retriever: this is the translation engine
    * None
* Game State: state machine to see if the application has started, user is playing, or user has finished playing
    * Label Retriever
    * Stateless
* My game's business logic: this is generic code that is not dependent upon the platform that I am running the code on. If you are familiar with N-Tier architecture then this concept already makes sense to you.
    * Game State
* My game's Unity logic: this is code that is dependent upon Unity assemblies that could not possibly survive or run without Unity being present.
    * Unity game engine    
    * My game's business logic

If I had Unity support I could simply depend on the business logic NuGet package and all of its dependencies would magically appear. Without Unity support I would need to keep track of each dependency that I had, and what dependencies those had, and recursively until I reach the end of the turtles. And they go all the way down. Yesterday I finally found a solution for this problem, enter [dotnet publish](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-publish) Now Visual Studio appears to have two publishing options and only using `dotnet publish` via command line, PowerShell, or NuGet console actually produces the results that I need.

Running this command will actually place your assembly and all of the assemblies that you are directly and indirectly dependent upon in a folder that you specify. This is exceptional! From the directory containing the solution I run the following command:

```
dotnet publish --output ..\..\Assets\References
```

This will generate a new build and then will place `Label Retriever`, `Game State`, `Stateless`, and `Business Logic` in the references folder for the Unity game logic to reference. One thing still bothers me about this though, I need to manually run this command in order to push the files. I cannot stand for such a thing, we must automate to purge ourselves of 400-step checklists for deployment.

## Making it automagical
In your project file you can specify certain events to run after a build has been complete. This is great when you want to have assemblies copied into dependent directories, which is the situation we are trying to solve. I updated my project file with the following target:

```csharp
  <Target Name="PostBuild" AfterTargets="PostBuildEvent">
    <Exec Command="dotnet publish .\LabyrinthBusinessLogic.csproj --output ..\..\Assets\References" />
  </Target>
```

Now with that bit of code, I sadly created a never ending loop that could run until the universe's heat death if I did not intervene. How did this happen? Well you and I were not paying attention when I mentioned that running `dotnet publish` would create a build and then copy the assemblies. So running this as a post build event would then create another build, and so forth until the nascent developer realizes their errors. Luckily there is a quick fix just add another flag to prevent building, `--no-build`. You can see the full command below.

```csharp
  <Target Name="PostBuild" AfterTargets="PostBuildEvent">
    <Exec Command="dotnet publish .\LabyrinthBusinessLogic.csproj --output ..\..\Assets\References --no-build" />
  </Target>
  ```

With each build of my business logic all of my dependencies will now be updated so that I can seamlessly switch to developing in the Unity-specific layer.

## Summary
Using `dotnet publish` as a post build event for .NET standard libraries it is trivial to manage NuGet packages as dependencies in a Unity-base application.

## Further reading
If you found this article interesting perhaps you would like to read how I [supported localization within Unity using C# Resources](https://kevinkabatra.wordpress.com/2019/11/10/supporting-localization-in-unity-using-c-resources/).

In addition, as with all of my projects, the work that this is based on is open-sourced and you can view it [here](https://github.com/kevinkabatra/UnityTutorialLabyrinth). In this repository you will see how I have implemented N-tier architecture, localization, and of course using NuGet.

## Consider sponsoring
If you found this helpful please consider sponsoring additional content. You can sign up to be a sponsor through [GitHub Sponsors](https://github.com/sponsors/kevinkabatra) and there are a few tiers of support that you can choose from. Any contributions would be greatly appreciated.