최신 dll적용방법
https://sourceforge.net/projects/mpv-player-windows/files/libmpv/
에서 다운로드, 이름이 바뀌기 때문에

winform 프로젝트 lib폴더에 libmpv-2.dll 추가 & 항상복사 옵션

mpvplayer.cs에서

		private readonly string[] possibleLibMpvPaths = new string[]
		{
			"libmpv-2.dll",
			@"lib\libmpv-2.dll",
			//"mpv-1.dll",
			//@"lib\mpv-1.dll"
		};
      
윈폼 프로젝트 속성에서 항상 64비트 설정


mpvfunction.cs에서
			DetachDestroy			= LoadFunction<MpvDetachDestroy>("mpv_destroy");
로 수정




====================================================================================================

# Mpv<span />.NET (lib)

[![Version](https://img.shields.io/nuget/v/Mpv.NET.svg?style=flat-square)](https://www.nuget.org/packages/Mpv.NET/)
[![Downloads](https://img.shields.io/nuget/dt/Mpv.NET.svg?style=flat-square)](https://www.nuget.org/packages/Mpv.NET/)

>Hello everyone!

>I developed this library a few years ago for another project of mine. Ever since I completed that project I have not been able to maintain this repository and investigate the issues. I know that a lot of you find this library useful and I want it to continue but I am not able to dedicate time towards it any more.

>If anyone is interested in maintaining this repository, reviewing pull requests and investigating issues. I would be more than happy to give you the right permissions to do that. I will also happily publish new version of the package on Nuget to make sure everyone can continue to enjoy the updates.

.NET embeddable video/media player based on [mpv](https://github.com/mpv-player/mpv) for WinForms and WPF

#### Player Features

* Looping
* Auto-play
* Frame stepping
* Asynchronous seeking
* Change playback speed
* Simple setup and usage
* Logging from mpv
* Add separate audio track
* Playlist - Load, Next, Previous, Move, Remove or Clear
* Optional [youtube-dl](https://rg3.github.io/youtube-dl/index.html) support to play videos from [hundreds of video sites](https://rg3.github.io/youtube-dl/supportedsites.html).
    * Change the desired video quality.

#### Notes:

* See [here](https://github.com/stax76/mpv.net) for Mpv.NET the C# media player based on mpv.
* Very little documentation has been written for the C API wrapper. Consult [client.h](https://github.com/mpv-player/mpv/blob/master/libmpv/client.h).
* The entirety of the mpv C API has not yet been implemented.

If you encounter any bugs or would like to see a feature added then please open an issue. Contributions are very welcome!

## Download

This package is available via [NuGet](https://www.nuget.org/packages/Mpv.NET).

## Prerequisites

### libmpv

To use the API wrapper (and player) you will need libmpv.

1. Download libmpv from [here](https://sourceforge.net/projects/mpv-player-windows/files/libmpv/).
   * Either "i686" if your app is 32-bit or "x86_64" if your app is 64-bit
2. Extract "mpv-1.dll" from the archive into your project.
    (A "lib" folder in your project is common practice)
3. Include the DLL in your IDE and instruct your build system to copy the DLL to output.
    * In Visual Studio:
        1. In Solution Explorer click the "Show All Files" button at the top. (make sure you have the correct project selected)
        2. You should see the DLL show up, right click on it and select "Include In Project".
        3. Right click on the DLL and select "Properties", then change the value for "Copy to Output Directory" to "Copy Always".
4. Done!

If you wish to compile libmpv yourself, there is a [guide](https://github.com/mpv-player/mpv/blob/master/DOCS/compile-windows.md) available in the mpv repository.

### youtube-dl

To enable youtube-dl functionality in the player you will need the youtube-dl executable and the ytdl hook script from mpv which allows mpv to communicate with youtube-dl.

1. Download the "youtube-dl.exe" executable from [here](https://rg3.github.io/youtube-dl/download.html).
2. Download the "ytdl_hook.lua" script from [here](https://github.com/mpv-player/mpv/blob/master/player/lua/ytdl_hook.lua).
3. Copy both files to your "lib" folder you made for libmpv.
4. Follow step #3 in the libmpv section but perform the steps on the "ytdl_hook.lua" script and "youtube-dl.exe" executable instead.
5. Near the start of the file, change the declaration of "ytdl" so it looks like this:
```lua
local ytdl = {
    path = "lib\\youtube-dl.exe",
    searched = false,
    blacklisted = {}
}
```
6. In your code, you will need to call the `EnableYouTubeDl` method on an instance of `MpvPlayer`.
    * If you placed your "ytdl_hook.lua" script somewhere other than the "lib" folder, you will need to pass the relative (to your apps executable) or absolute path of the script to `EnableYouTubeDl`.
7. Done!

To keep youtube-dl functionality working:
* Regularly update the "youtube-dl.exe" executable with latest version.
* Regularly update the "ytdl_hook.lua" script with the latest version.

If you have any doubts or questions regarding this process, please feel free to open an issue.

## Player

This player was designed to work on Windows and tested in WPF and WinForms. Not tested on other platforms.

To overlay controls over the top of the player please start with this [Stack Overflow post](https://stackoverflow.com/questions/5978917/render-wpf-control-on-top-of-windowsformshost).

If you're looking for a media player UI, I'd recommend [MediaPlayerUI.NET](https://github.com/mysteryx93/MediaPlayerUI.NET). :)

### Initialisation

`MpvPlayer` provides 4 constructors:
1. `MpvPlayer()`
2. `MpvPlayer(string libMpvPath)`
3. `MpvPlayer(IntPtr hwnd)`
4. `MpvPlayer(IntPtr hwnd, string libMpvPath)`

Constructors #1 and #2 let mpv create it's own window and will not embed it.

Constructors #3 and #4 allow you to specify a `hwnd` parameter which should be the handle to the host control where you want to embed the player.

For the constructors where `libMpvPath` is not included, the player attempts to load libmpv from: (in order)
* LibMpvPath property
* "mpv-1.dll"
* "lib\mpv-1.dll"

### WPF

Since WPF doesn't keep traditional handles to controls we will need to use a `System.Windows.Forms.Control` object (E.g. `Panel` or nearly any other WinForms control) to host the mpv instance.

First you will need to add a reference to `System.Windows.Forms` in your project:
* In Visual Studio this can be achieved so:
    * Right click "References" in your project and click "Add Reference".
    * Navigate to Assemblies, find `System.Windows.Forms`, make sure it's ticked and click "OK".

Next, you will need to reference the `System.Windows.Forms` namespace in XAML:

```xml
<Window ...
        xmlns:windowsForms="clr-namespace:System.Windows.Forms;assembly=System.Windows.Forms"
        ...>
```

Then create a `WindowsFormsHost` with a `Panel` from WinForms:

```xml
<WindowsFormsHost>
    <windowsForms:Panel x:Name="PlayerHost" />
</WindowsFormsHost>
```

If `WindowsFormHost` isn't found you will need to add a reference to `WindowsFormsIntegration` to your project:
* Just as above: this can be achieved in Visual Studio by:
    * Right clicking "References" in your project and clicking "Add Reference".
    * Navigating to Assemblies, finding `WindowsFormsIntegration`, making sure it's ticked and clicking "OK".

In your `.xaml.cs` file create a field/property of type `MpvPlayer` and initialise it using one of the constructors outlined above.

In this example we called our `Panel` object `PlayerHost` so for the `hwnd` parameter in the constructor you would pass in `PlayerHost.Handle` like so:
```csharp
player = new MpvPlayer(PlayerHost.Handle);
```

See [Mpv.NET.WPFExample](https://github.com/hudec117/Mpv.NET/tree/master/src/Mpv.NET.WPFExample) project for a basic example.

### WinForms

You can use any WinForms control, just pass the `Handle` property to the `MpvPlayer` constructor and you're done! Easy.

See [Mpv.NET.WinFormsExample](https://github.com/hudec117/Mpv.NET/tree/master/src/Mpv.NET.WinFormsExample) project for a basic example.

### youtube-dl settings

You have the option to set the desired quality of the media you're trying to play using youtube-dl.
This can be changed by setting the `YouTubeDlVideoQuality` property on an instance of `MpvPlayer`.
Note: this will take effect on the next call of `Load`.

## API

This "API" is a wrapper around the mpv C API defined in [client.h](https://github.com/mpv-player/mpv/blob/master/libmpv/client.h) and is utilised by the player.

### Mpv

Simplest way to create an instance of the mpv .NET API and load libmpv:
```csharp
// Relative path to the DLL.
var dllPath = @"lib\mpv-1.dll";

using (var mpv = new Mpv(dllPath))
{
    // code
}
```

This will provide a friendly interface to the mpv C API and start an event loop which will raise events accordingly.

### MpvFunctions

If you are looking for a more direct approach to the C API you can create an instance of MpvFunctions which allows you to execute the loaded delegates directly:
```csharp
var dllPath = @"lib\mpv-1.dll";

using (var mpvFunctions = new MpvFunctions(dllPath))
{
    // code
}
```

Be aware, this method does not raise events and an event loop would have to  be manually implemented.

### MpvEventLoop

If you are looking that start an event loop you will need to create an instance of MpvFunctions, create and initialise mpv, create an instance of MpvEventLoop and start it:
```csharp
var dllPath = @"lib\mpv-1.dll";

// Create an instance of MpvFunctions.
var functions = new MpvFunctions(dllPath);

// Create mpv
var handle = functions.Create();

// Initialise mpv
functions.Initialise(handle);
    
// Create an instance of MpvEventLoop, passing in a callback argument
// which will be invoked when an event comes in.
using (var eventLoop = new MpvEventLoop(callback, handle, functions))
{
    // Start the event loop.
    eventLoop.Start();
}
```

The code above does not contain any error checking, most of the mpv functions return an MpvError which indicates whether an error has occured.

## Licensing

The libmpv C API *specifically* is licensed under [ICS](https://choosealicense.com/licenses/isc/), this means that a wrapper such as this can be licensed under [MIT](https://choosealicense.com/licenses/mit/).

The rest of libmpv is licensed under [GPLv2](https://choosealicense.com/licenses/gpl-2.0/) by default, which means that any work utilising this wrapper in conjunction with libmpv is subject to GPLv2, unless libmpv is compiled using [LGPL](https://choosealicense.com/licenses/lgpl-2.1/).

In simple terms, once you use the "libmpv" files (DLL) you downloaded, your application must be licensed under GPLv2.

See [here](https://github.com/mpv-player/mpv#license) for more information.
