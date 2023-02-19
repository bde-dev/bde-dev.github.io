---
## Linking an unmanaged C++ DLL into a single C# exe with Costura.Fody

[Overview](#overview) <br>
[isGenuinewindows() C#](#isgenuinewindows-with-csharp-attempt) <br>
[isGenuineWindows() C++](#isgenuinewindows-with-cpp-attempt) <br>
[Next Hurdle](#next-hurdle-distribution) <br>
[Setting up the linkage - TL;DR with pictures](#setting-up-the-linkage)<br>

### _**Story Time**_

#### Overview
I decided to create a C# .NET 6.0 x64 Windows Forms app in Visual Studio 2022 to help me with my work. Its function was to deploy software products, based on a clean Windows 10 image that auto-licenses when the first network it sees is the internet via the corporate LAN.

This post describes the steps taken to get to where the app has (hypothetically) licensed the windows 10 installation, and it double checks the activation status is valid.
<br>

#### IsGenuineWindows() with Csharp attempt
To do that, the first option I looked at was doing it **_natively_** using C# assemblies, so I looked into a **WMI** solution, using [System.Management](https://www.nuget.org/packages/System.Management). I downloaded it as a NuGet package and added a reference to the project. This brought lots of trouble, because when code snippets didn't work, I couldn't debug the generic windows errors when I called the [Get()](https://learn.microsoft.com/en-us/dotnet/api/system.management.managementobjectsearcher.get?view=dotnet-plat-ext-7.0) method on the [ObjectSearcher](https://learn.microsoft.com/en-us/dotnet/api/system.management.managementobjectsearcher?view=dotnet-plat-ext-7.0) class instance.
<br>

#### IsGenuineWindows() with CPP attempt
This is why I decided to go for a more **_modular_** solution, by creating a **C++** WindowsActivation.dll containing a function that can check the activation status of the host Windows installation.

>Naturally, this required the [vcredist_x64](https://learn.microsoft.com/en-US/cpp/windows/latest-supported-vc-redist?view=msvc-170) installed on the windows 10 image.

I started by making a simple C++ console app that checked the activation status of a host machine; this worked on my activated Windows 10 Pro work laptop and correctly identified an unlicensed fresh Windows 10 image.

Now that I have working code that verifies Windows 10 license status, I configured the C++ project to build as a DLL, copied the DLL over to the output folder of the C# forms app, wrote the required DLL import code, and set the app to call the function in the DLL on a button press.

**It worked!**
<br>

#### Next hurdle: distribution
Now I have an issue where I need to distribute a DLL **_in addition_** to a Deploy.exe in the start-up folder. I decided I would like a cleaner solution. I Googled this problem finding code snippets where the raw bytes inside the DLL are being read in to a stream and parsed into workable C# code. I did not like that idea. <br> <br>
One result mentioned [Costura.Fody](https://www.nuget.org/packages/Costura.Fody/5.8.0-alpha0098), a NuGet package that compiles assembles as embedded resources, allowing those resources to be built into the .exe, which is exactly what I need.
<br>

#### Setting up the linkage

- Both [FODY](https://www.nuget.org/packages/Fody) and [Costura.Fody](https://www.nuget.org/packages/Costura.Fody/5.8.0-alpha0098) NuGet packages are required, which can be installed via the NuGet package manager in Visual Studio 2022, _provided the package manager component is installed_.
![image tooltip here](/screenshots/NuGet-package-manager.jpg)
<br>

- Did some skimming of the [README](https://github.com/Fody/Costura/blob/develop/readme.md) on GitHub, and landed on a very simple FodyWeavers.XML:
{% highlight xml %}
<Weavers xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="FodyWeavers.xsd">
	<Costura CreateTemporaryAssemblies='true'
			 IncludeDebugSymbols='false'>
		<Unmanaged64Assemblies>
			WindowsActivation
		</Unmanaged64Assemblies>
	</Costura>
</Weavers>
{% endhighlight %}
<br>

 - Made a **Costura64** folder (Costura32 folder for 32bit DLLs) to put DLL file in. _This wasn't in the README_.
![image tooltip here](/screenshots/costura64_folder.jpg)
<br>

- Then I needed to go to the **properties** of the WindowsActivation.dll inside the Costura64 folder and set the **Build Action** property to **Embedded Resource**.
![image tooltip here](/screenshots/dll_props.jpg)
<br>

- Combine this with my Visual Studio 2022 publish profile:
{% highlight xml %}
<Project>
  <PropertyGroup>
	<Configuration>Release</Configuration>
	<Platform>Any CPU</Platform>
	<PublishDir>C:\Dev\C#</PublishDir>
	<PublishProtocol>FileSystem</PublishProtocol>
	<_TargetId>Folder</_TargetId>
	<TargetFramework>net6.0-windows</TargetFramework>
	<RuntimeIdentifier>win-x64</RuntimeIdentifier>
	<SelfContained>false</SelfContained>
	<PublishSingleFile>true</PublishSingleFile>
	<PublishReadyToRun>true</PublishReadyToRun>
  </PropertyGroup>
</Project>
{% endhighlight %}

And now When I publish this project, I get a **_single .exe_** for my forms app that, on a button press, correctly identifies whether the host Windows system is activated!

Hopefully this helps a few people because it took me a few days of scraping around to find the correct info.

