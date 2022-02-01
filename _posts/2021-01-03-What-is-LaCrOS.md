---
layout: post
title: Introduction to LaCrOS
---

Happy new year to All !!! 

Igalia has been collaborating with Google on the LaCrOS project for over a year and has presented updates of the LaCrOS project in the last two BlinkOns.

See ours lightning talk during [BlinkOn13](https://www.youtube.com/watch?v=6Wgh5x7mLSM&t=1621s) , and our update from [BlinkOn14](https://www.youtube.com/watch?v=VrEP7SPfQVM&t=2514s)

## What is LaCrOS ?

For the unfamiliar , the [LaCrOS](https://source.chromium.org/chromium/chromium/src/+/main:docs/lacros.md) project decouples the Chrome browser from the Chrome OS window manager and system UI. The name comes from **L**inux **A**nd **C**h**R**ome**OS**. 

But, why do we need to do that ? Let's jump to more details.

## Why LaCrOs ?

Previously, ChromeOS and Chrome Browser shared the same window manager and system UI binaries, making it difficult for Chrome Browser alone to be updated on Chromebook devices. 

This means, when a ChromeOS device is updated, only then the Chrome Browser is updated. It might be frustrating for someone who frequently switches from the latest desktop browser to a browser on ChromeOS.

LaCrOS is Google's solution to this problem. It allows browser on ChromeOS to update independently of the operating system. Now, old chromebook devices also can have the latest Chrome Browser.

> Separating the browser from the OS itself allows the end user to update the browser independently as we do for other devices like desktop, mobile.

On Chrome OS, the system UI (including the Ash Window Manager, login screen, etc) and the Web Browser are included in the same binary.  LaCrOS separates this functionality into two binaries. Both of these binaries are built from the same chromium git repository, but they might have different versions. This means, Web Browser and ChromeOS can have different versions and will be updated separately.

Some of the key LaCrOS components are ChromeOS’ built-in Wayland Compositor ([Exo](https://source.chromium.org/chromium/chromium/src/+/main:components/exo/)) and the native Linux Wayland support (on top of [Ozone](https://source.chromium.org/chromium/chromium/src/+/main:docs/ozone_overview.md)). As part of the project, Linux [Wayland](https://chromium.googlesource.com/chromium/src/+/main/ui/ozone/platform/wayland/README.md) is being enhanced to implement ChromeOS specific functionality so that the a newly created browser acts like a Web Browser under ChromeOS.

This new browser which uses Linux Wayland support underneath to communicate with ChromeOS is called a LaCrOS(**L**inux **A**nd **C**h**R**ome**OS**).

In terms of wayland terminology, a LaCrOS-Chrome is a wayland-client and Exo is a wayland-server. 

LaCrOS development involves both client and server code changes. ChromeOS implements handlers for requests coming from LaCrOS Browser, and LaCrOS Browser implements handlers for events coming from ChromeOS.

## What is Ash-Chrome ?

Ash-Chrome is the default browser of ChromeOS which is tightly coupled with ChromeOS. For client-server architecture, Ash-Chrome is often referred as a server and used as a synonymous to ChromeOS or Exo.

LaCrOS client request a information from a Ash-Chrome server.

## What is Exo ?

[Exo](https://source.chromium.org/chromium/chromium/src/+/main:components/exo/)(or Exosphere) is a wayland server implementation. LaCrOS is one of the client of Exo. There are other clients of Exo which includes ARC++(Android Apps on ChromeOS) and Crostini (Linux apps on Chrome OS).

## A simplified LaCrOS Architecture

![](https://notes.igalia.com/uploads/b15227e6fe0fbdf1d7661a302.jpg)

In the above diagram, LaCrOS Browser communicates with ChromeOS(Ash-Chrome) using a wayland protocol. LaCrOS uses various wayland protocols
* [Official](https://source.chromium.org/chromium/chromium/src/+/main:third_party/wayland-protocols/) wayland protocols - These are developed by the Wayland community and well known desktop environments. 
* [Non-official](https://source.chromium.org/chromium/chromium/src/+/main:components/exo/wayland/protocol/) wayland protocols - These protocols are specific to ChromeOS.

Exo is a wayland server and it supports both the official and non-official wayland protocols. Just for an example, [aura_shell](https://source.chromium.org/chromium/chromium/src/+/main:components/exo/wayland/protocol/aura-shell.xml) is a non-official protocol which allows the LaCrOS browser to support the ChromeOS specific features like snapping a window and many more.

Implementation of LaCrOS is inspired from Chrome on desktop linux. My colleague [Maksim](mailto:msisov@igalia.com) in his talk [Waylandifying Chromium](https://www.youtube.com/watch?v=8t4DUIZ94pk) has explained how a Wayland is used in Chromium.

Next obvious question would be how to build and run the LaCrOS Browser and contribute to feature LaCrOS development.

## Build & Run
Refer to [Chromium OS on Linux (linux-chromeos)](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/docs/chromeos_build_instructions.md#chromium-os-on-linux-linux_chromeos) before building code

You can download a [GN](https://www.chromium.org/developers/gn-build-configuration/) args necessary to build a LaCrOS on linux-desktop from an already running `linux-lacros-rel` build bot.

```
chromium/src$tools/mb/mb.py gen -m tryserver.chromium.chromiumos -b \'linux-lacros-rel' out/linux-lacros-rel
```

The above command downloads the GN args and writes the file name args.gn in the output directory `out/linux-lacros-rel`.

```
# file: args.gn
also_build_ash_chrome = true
blink_enable_generated_code_formatting = false
chromeos_is_browser_only = true
dcheck_always_on = true
is_component_build = false
is_debug = false
symbol_level = 0
target_os = "chromeos"
use_goma = true
```
`also_build_ash_chrome = true` tells a [ninja](https://chromium.googlesource.com/chromium/src.git/+/56.0.2924.56/docs/ninja_build.md) to build Ash-Chrome(wayland server) along with LaCrOS Browser(wayland client)

* ### Build
To build the code run the below command from your chromium source directory.
```
chromium/src$autoninja -C out/linux-lacros-rel/ chrome all -j500
```
After build, Ash-Chrome is located at location `out/linux-lacros-rel/ash_clang_x64/test_ash_chrome` and LaCrOS Browser is located at location `out/linux-lacros-rel/chrome`.

* ### Run
To run the use the below command
```
chromium/src$XDG_RUNTIME_DIR=/tmp/ash_chrome_xdg_runtime out/linux-lacros-rel/ash_clang_x64/test_ash_chrome --user-data-dir=/tmp/ash-chrome --enable-wayland-server --no-startup-window --enable-features=LacrosSupport --lacros-chrome-path=$PWD/out/linux-lacros-rel --enable-logging=stderr
```
It launches the ChromiumOS emulator with support for LaCrOS browser
The instructions to run are also publicly available on some issues on [crbug.com.](https://bugs.chromium.org/p/chromium/issues/list?q=%22--lacros-chrome-path%22&can=1) If you want more information about the different command line parameters, you can check bugs.

## LaCrOS window during BlinkOn 13

![](https://notes.igalia.com/uploads/b15227e6fe0fbdf1d7661a304.png)

In the above image, the window of LaCrOS-Chrome looks visually different from the Ash-Chrome. If you look carefully, you will notice that the scrollbar and the browser window's frame appear differently and LaCrOS-Chrome does not have drop shadows.

## LaCrOS window during BlinkOn 14

![](https://notes.igalia.com/uploads/b15227e6fe0fbdf1d7661a305.png)

This image depicts the state of LaCrOS Browser at the time of [the BlinkOn 14 conference](https://www.chromium.org/events/blinkon-14). As you can see both browsers look almost identical. 

## Igalia's contribution to LaCrOS in 2021
To achieve visual parity between both browsers, our team at Igalia has worked to support various features and bugs related to
- Rounded corners for browser window
- Scrollbars style, thickness and fade behavior
- Resizing behavior
- Shadows (Drop shadow for window & menu, resizer shadow)
- Menus (Right-Click menus, Three-Dot menus)
- Tooltips
- Tablet mode
- Window Snapping
- HiDPI/Mixed DPI
- Touch based window dragging support
- Multiple display
- Scaling
- Enhanced tab dragging


LaCrOS Browser evolution in the last one year can be seen in this video. 

<figure class="video_container">
  <iframe src="https://www.youtube.com/embed/LWF-aCcWyLI" frameborder="0" allowfullscreen="true"> </iframe>
</figure>

This video demonstrated the visual similarities between the Ash-Chrome and LaCrOS-Chrome. You can notice the similarities for
 - Drop shadows for a browser window frame
- Resizer shadows after hovering a mouse on browser window edges
- Right-Click menus, Three-Dot menus look and feel is same
- Scrollbars
- Window snapping for LaCrOS Browser window
- Workspace resizing

## And more ...
In addition to what has been commented above, we recently completed the **Multi Desk** support and **WebUI Tab Strip** features:
- Multi-Desk 
This feature allows the user to add desks to organize multiple windows and multi-task.
The LaCrOS-Chrome browser supports this feature by adding a request to a aura_shell wayland protocol. My colleague [Minju Kim](mailto:mkim@igalia.com) has contributed to an implementation of this feature.
See the working of the Multi-Desk in LaCrOS in the video below.

<figure class="video_container">
  <iframe src="https://www.youtube.com/embed/BuyocSkrbl4" frameborder="0" allowfullscreen="true"> </iframe>
</figure>

- WebUi Tab Strip
This feature adds a new tab bar to the browser, featuring page thumbnail previews and a touch friendly UI. My colleague [Antonio](mailto:tonikitoo@igalia.com) has contributed to an implementation of this feature.

Last but not least, our Chromium team is also working in parallel on fixing several window manager integration issues as part of this work, so please stay tuned for more updates on our efforts related to LaCrOS development :)
