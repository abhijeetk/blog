---
layout: post
title: Building nwjs on Windows7 64 bit 
---
##Install depot_tools : ##
- Download code

~~~~
$ git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
~~~~

- Add depot_tools to your PATH:

~~~~
$ export PATH=$PATH:~/depot_tools (MAC/Linux)
~~~~

On Windows, add depot_tools to PATH enviorment variable.

##Get nwjs source code : ##
- Create source directory and download configuration .gclient file.

~~~~
    node-webkit
    cd node-webkit
    touch .gclient
    vi .gclient
~~~~
- Now edit the file .gclient, and paste this into it:

~~~~
    solutions = [
       { "name"        : "src",
         "url"         : "https://github.com/nwjs/chromium.src.git@origin/nw12",
         "deps_file"   : "DEPS",
         "managed"     : True,
       
         "custom_deps" : {
           "src/third_party/WebKit/LayoutTests": None,
           "src/chrome_frame/tools/test/reference_build/chrome": None,
           "src/chrome_frame/tools/test/reference_build/chrome_win": None,
           "src/chrome/tools/test/reference_build/chrome": None,
           "src/chrome/tools/test/reference_build/chrome_linux": None,
           "src/chrome/tools/test/reference_build/chrome_mac": None,
           "src/chrome/tools/test/reference_build/chrome_win": None,
         },
         "safesync_url": "",
       },
    ]
~~~~

- Environment variables

~~~~
    DEPOT_TOOLS_WIN_TOOLCHAIN=0 
    DXSDK_DIR=C:\Program Files (x86)\Microsoft DirectX SDK (June 2010)\  
    GYP_GENERATOR=ninja,msvs
    GYP_GENERATORS=ninja
    GYP_DEFINES=nwjs_sdk=1 disable_nacl=0 proprietary_codecs=1
    GYP_MSVS_VERSION=2013
~~~~

Add machine architecture information:
For 64 bit OS :

~~~~
    $GYP_DEFINES=host_arch=x64 target_arch=x64 
~~~~

##Sync Code##
~~~~
    $node-webkit>gclient sync
~~~~

##Build##
~~~~
    $ninja -C out/Release_x64 nw -j16
~~~~

##Clean code##
~~~~
    $ninja -C out/Release_x64 -t clean
~~~~