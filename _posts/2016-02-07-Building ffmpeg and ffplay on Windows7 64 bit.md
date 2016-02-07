---
layout: post
title: Building ffmpeg and ffplay on Windows7 64 bit
---
##Prerequisite for ffmpeg##
- Install Msys64 on yourmachine. (msys2-x86_64-20150916.exe) 

- Open VS2013 x64 Native Tools Command Prompt
C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\Tools\Shortcuts

- Run C:/msys64/mingw64_shell.bat in VS2013 x64 Native Tools Command Prompt
 
- Check current compiler : It should be pointing to VS2013 compiler
 
- Check current linker, if it is referring to /usr/bin/link then we have to change it by renaming link.exe. Rename C:\msys64\usr\bin\link.exe to C:\msys64\usr\bin \link_old.exe

- YASM : Download YASM from http://www.tortall.net/projects/yasm/releases/yasm-1.3.0-win64.exe
    - Rename yasm-1.3.0-win64.exe to yasm.exe
    - Place yasm.exe somewhere in your PATH environment variable or Add it to C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin

- Make sure to use mingw64_shell.bat to have the correct MinGW-w64 environment. Run following commands in MinGW-w64 to install dependencies.

~~~~
    #normal msys2 packages
    pacman -S make pkgconfdiffutils
    # mingw-w64 packages and toolchains
    pacman -S mingw-w64-x86_64-yasm mingw-w64-x86_64-gcc mingw-w64-x86_64-SDL
~~~~

##Download ffmpeg code##

~~~~
    mkdir ffmpeg
    cd ffmpeg
    git clone https://github.com/FFmpeg/FFmpeg.git
~~~~

##Build instructions for ffmpeg## 
Ffmpeg can be built directly using following commands. If you wish to compile shared libraries, add --enable-shared to your configure options.

~~~~
    ./configure --toolchain=msvc
    make
    make install
~~~~

##Build instructions for ffplay##

- Download SDL from http://www.libsdl.org/release/SDL-devel-1.2.15-VC.zip
- Extract it and copy 
    - SDL-devel-1.2.15-VC\SDL-1.2.15\include\*.*  to C:\msys64\mingw64\include\SDL
        - Click -> Copy and Replace
    - SDL-devel-1.2.15-VC\SDL-1.2.15\lib\x64 to
C:\msys64\mingw64\lib

- Open file C:\msys64\mingw64\bin\sdl-config and modify following lines

~~~~bash 
        --libs)
  echo -L${exec_prefix}/lib -lSDLmain -lSDL
        ;;
      --static-libs)
  #    --libs|--static-libs)
  echo -L${exec_prefix}/lib -lSDLmain -lSDL
        ;;
~~~~

- Run following commands : 

~~~~
./configure --enable-ffplay --toolchain=msvc --disable-shared --enable-static --enable-sdl --extra-ldflags="-Wl,-add-stdcall-alias" --enable-memalign-hack --disable-ffmpeg --pkg-config=sdl-config
~~~~