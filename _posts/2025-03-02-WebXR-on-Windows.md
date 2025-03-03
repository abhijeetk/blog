---
layout: post
title: WebXR on Windows
---
## WebXR on Windows

It started with my need to debug Chromium's implementation of OpenXR. I wanted to understand how Chromium interfaces with OpenXR APIs. However, I noticed that only the Android and Windows ports of Chromium currently support OpenXR bindings. Since I needed to debug a desktop implementation, Windows was the only viable option. Additionally, I did not have access to a physical XR device, so I explored whether a simulator or emulator environment could be used to test WebXR support for websites.

### Chromium OpenXR Implementation
Chromium's OpenXR implementation, which interacts with the platform-specific OpenXR runtime, is located in the `device/vr/` directory. WebXR code interacts with this `device/vr/` OpenXR implementation, which abstracts WebXR features across multiple platforms.

```
WebXR ---> device/vr/ ---> OpenXR runtime
```

### Installing OpenXR Runtime
To run OpenXR on Windows, you need to install an OpenXR runtime. You can download and install **OpenXR Tools for Windows Mixed Reality** from the Microsoft App Store:

[![OpenXR Tools for Windows Mixed Reality](https://store-images.s-microsoft.com/image/apps.49732.14036023774986754.8b104295-c4f4-48ad-b4c1-925b8e216e67.9002afb2-b23d-4d2f-b55f-7fe68687f8d5?h=720)](https://apps.microsoft.com/store/detail/openxr-tools-for-windows-mixed-reality/9N5CVVL23QBT)

If it is not available on your machine, you can enable it from the **OpenXR Runtime** tab in the application.

### Installing Microsoft Mixed Reality Simulator
To set up a simulated environment for WebXR testing, follow these steps:

1. Install **Mixed Reality Portal** from the Microsoft App Store.
2. Follow the official Microsoft guide on enabling the Mixed Reality simulator:
   [Using the Windows Mixed Reality Simulator](https://learn.microsoft.com/en-us/windows/mixed-reality/develop/advanced-concepts/using-the-windows-mixed-reality-simulator#enabling-the-windows-mixed-reality-simulator)

If you encounter hardware compatibility errors, refer to the troubleshooting steps in the guide.

### Connecting Chromium to OpenXR Implementation
Chromium provides a flag to select the OpenXR implementation.

1. Open Chrome and navigate to:
   ```
   chrome://flags/#webxr-runtime
   ```
2. Set the flag to `OpenXR`.

This enables Chromium to use the OpenXR runtime for WebXR applications.

### Launch WebVR application
Launch chromium and Open : https://immersive-web.github.io/webxr-samples/immersive-vr-session.html

![output](https://matrix.igalia.com/_matrix/media/v3/download/igalia.com/PBDrNUfMiATtyGJiThKkfdMz?allow_redirect=true)

### CallStack

