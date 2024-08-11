---
layout: post
title: Exploring sandboxing in CEF and CEFSharp for Windows platform
---
Sandboxing is a critical feature for ensuring the security and stability of applications that embed web content. In this blog, we will delve into how sandboxing is achieved in the Chromium Embedded Framework ([CEF](https://bitbucket.org/chromiumembedded/cef/src/master/)). All analysis in this blog is performed on Windows operating system.


### CEFClient and CEFSharp
The base CEF framework includes support for the C and C++ programming languages. Also it provides a test applications to demonstrate its functionality. [CEFClient](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefclient/), a one of the C++ test application, serves this purpose. 

CEF provides CAPI bindings to enable application development across various programming languages. For integrating the CEF framework with .NET, you should explore [CEFSharp](https://github.com/cefsharp/CefSharp). It is a C# application that leverages the CEF open source project, allowing developers to create applications in C#.

```
CEFSharp(C#)
            \  CAPI 
              -------> CEF(C++) ------> Chromium(C++)
            /
CEFClient(C++)
```
---

### What is the sandbox?
As mentioned in [documentation](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/design/sandbox_faq.md#what-is-the-sandbox) of sandbox in Chromium repository,
> The sandbox is a C++ library that allows the creation of sandboxed processes — processes that execute within a very restrictive environment. The only resources sandboxed processes can freely use are CPU cycles and memory. For example, sandboxes processes cannot write to disk or display their own windows. What exactly they can do is controlled by an explicit policy. Chromium renderers are sandboxed processes.
---

### Understanding Sandboxing in CEF

Since CEF uses Chromium internally, we decided to investigate how sandboxing is implemented by examining Chromium's source code. Our findings revealed that the sandbox is designed to be versatile, with no hard dependencies on the Chromium browser itself, making it usable with other applications.

CEF prepares a static [library](https://bitbucket.org/chromiumembedded/cef/src/69b884d39cbcb6497187a65d1fac6782ef823040/BUILD.gn#lines-1181) using the existing [sandbox](https://source.chromium.org/chromium/chromium/src/+/main:sandbox/win/src/) implementation in the Chromium repository and links it to the main application. This process enables the sandbox functionality in CEF applications. For more detailed information, refer to the [sandbox_win.cc](https://bitbucket.org/chromiumembedded/cef/src/master/libcef_dll/sandbox/sandbox_win.cc) file in the CEF repository.

```
if (is_win) {
  static_library("cef_sandbox") {
    sources = [ "libcef_dll/sandbox/sandbox_win.cc" ]
    include_dirs = [ "." ]
    deps = [ "libcef/features", "//sandbox" ]
  }
}
```

---

### Implementing Sandboxing in CEFClient

CEFClient achieves a sandboxing by linking to cef_sandbox library
```
  executable("cefclient") {
      deps += [
        ":cef_sandbox",
      ]
```

In the CEFClient application, we need to create a sandbox object and pass it to CEF, which then forwards it internally to Chromium to configure the sandbox.
```
  CefScopedSandboxInfo scoped_sandbox;
  sandbox_info = scoped_sandbox.sandbox_info();
  ...
  // Initialize the CEF browser process.
  if (!context->Initialize(main_args, settings, app, sandbox_info)) {
    return 1;
  }
```

The following call stack illustrates how CEFClient code calls into CEF, ultimately transitioning into the Chromium environment to configure the sandbox.

![]({{ site.baseurl }}/images/Exploring-sandboxing-in-CEF-and-CEFSharp/bd290d80-d51f-4772-8e24-dc57ee73ba47.png)


The CEF test application (CEFClient) successfully launches a sandboxed CEF, which is verified by loading the ```chrome://sandbox``` URL.

![]({{ site.baseurl }}/images/Exploring-sandboxing-in-CEF-and-CEFSharp/cb0b3336-4af7-49b1-82ed-2e1abccc2e38.png)

---

### Why C# Applications Cannot Use Sandboxed CEF ?

As we have seen above, for C++ applications it is possible to use sandboxing but C# application can not use sandboxed CEF. But why ? 

To achieve effective sandboxing in your CEF application, ensure that you link the `cef_sandbox.lib` static library. This linking must be done specifically to the main process of the application. While C++ applications can directly link static libraries without issue, C# applications do not support static linking of libraries.

> ```Static library means that the library is going to be merged with application. This concept doesn't exist in .NET as .NET supports only dynamic link libraries```

If you attempt to create a DLL for the cef_sandbox code and link it dynamically to your application, the Chromium sandbox code will detect this and fail with the error [SBOX_ERROR_INVALID_LINK_STATE](https://source.chromium.org/chromium/chromium/src/+/main:sandbox/win/src/sandbox_types.h;l=154?q=SBOX_ERROR_INVALID_LINK_STATE). Therefore, static linking of the sandboxing code to the main application is mandatory for sandbox functionality.


```
  // Attempt to start a sandboxed process from sandbox code hosted not within
  // the main EXE. This is an unsupported operation by the sandbox.
  SBOX_ERROR_INVALID_LINK_STATE = 64,
```
---
### Conclusion
While sandboxing is a powerful feature available for C++ applications using CEF, it is not feasible for C# applications due to the inherent differences in how these languages operate within the system. The managed environment of C# introduces limitations that make it impossible to integrate the same level of sandboxing as in C++. Consequently, C# applications cannot leverage sandboxed CEF using the existing setup.

---
### Future study
To enable sandboxing in C# applications, we may need to modify the .NET [runtime](https://github.com/.NET/runtime) (which is written in C++) to support sandboxing. This would involve linking the cef_sandbox library directly to the runtime. The .NET Core runtime includes several executables/libraries that act as the main entry point for code execution, typically referred to as the "[host](https://github.com/.NET/runtime/blob/main/docs/design/features/host-components.md)". These host executables can be customized as needed.

For example, when you run the apphost executable, it initializes the .NET Core runtime and starts your application. Creating a sandbox object in the apphost and passing it to the application might help address sandboxing for C# applications. However, this approach has not been tested, so its effectiveness cannot be confirmed without further exploration and modification of the C# language runtime.

---

**References**
- [https://stackoverflow.com/questions/1590337/using-the-google-chrome-sandbox](https://stackoverflow.com/questions/1590337/using-the-google-chrome-sandbox)
- [https://chromium.googlesource.com/chromium/src/+/HEAD/docs/design/sandbox.md](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/design/sandbox.md)
- [https://chromium.googlesource.com/chromium/src/+/HEAD/docs/design/sandbox_faq.md](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/design/sandbox_faq.md)