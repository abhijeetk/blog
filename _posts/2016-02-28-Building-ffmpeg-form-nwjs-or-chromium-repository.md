---
layout: post
title: Building ffmpeg form nwjs/chromium repository
---

- Open VS2013 x64 Native Tools Command Prompt
C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\Tools\Shortcuts

- Open C:\cygwin64\Cygwin.bat in VS2013 x64 Native Tools Command Prompt and make sure that compiler and linker are refering to VS2013 64 bit executables as shown below.[Assuming you have cygwin installed on your Windows machine]

![alt text]({{ site.baseurl }}/images/Building-ffmpeg-form-nwjs-or-chromium-repository/image001.png "{{ page.title }}")

- Set following environment variables

~~~~ bash
    set GYP_DEFINES=host_arch=x64 target_arch=x64 nwjs_sdk=1 disable_nacl=0 proprietary_codecs=1 branding=Chrome
~~~~

![alt text]({{ site.baseurl }}/images/Building-ffmpeg-form-nwjs-or-chromium-repository/image002.png "{{ page.title }}")

- Go to Ffmpeg source code location : 

~~~~ bash
    <node-webkit-dir>\src\third_party\ffmpeg
~~~~

- **[APPLY PATCH to file ]** <ffmpeg_src>/chromium/scripts/build_ffmpeg.py

Add support of mxf by enabling it as decoder, demuxer and parser

~~~~diff
-      '--enable-decoder=pcm_s16be,pcm_s24be,pcm_mulaw,pcm_alaw',
-      '--enable-demuxer=ogg,matroska,wav',
-      '--enable-parser=opus,vp3,vorbis,vp8',

+      '--enable-decoder=pcm_s16be,pcm_s24be,pcm_mulaw,pcm_alaw,mxf',
+      '--enable-demuxer=ogg,matroska,wav,mxf',
+      '--enable-parser=opus,vp3,vorbis,vp8,mxf',
~~~~

- **[APPLY PATCH to file ]** : <ffmpeg_src>/chromium/scripts/build_ffmpeg.py

Add support of mxf by enabling it as decoder, demuxer and parser

~~~~diff    
       ['chromeos == 1', {
         'ffmpeg_branding%': '<(branding)OS',
       }, {  # otherwise, assume Chrome/Chromium.
-        'ffmpeg_branding%': '<(branding)',
+         'ffmpeg_branding%': 'Chrome',
       }],
     ],
~~~~


- Build standalone ffmpeg code for windows 64 bit.

![alt text]({{ site.baseurl }}/images/Building-ffmpeg-form-nwjs-or-chromium-repository/image003.png "{{ page.title }}")

- Build will generate following directory **build.x64.win**<ffmpeg_src> folder.

- Copy config and from **build.x64.win**:

~~~~ bash
ffmpeg_src>./chromium/scripts/copy_config.sh
~~~~

Regenerate ffmpeg gyp files :

~~~~ bash
ffmpeg_src>python chromium/scripts/generate_gyp.py
~~~~

![alt text]({{ site.baseurl }}/images/Building-ffmpeg-form-nwjs-or-chromium-repository/image004.png "{{ page.title }}")

Following files get modified after running above commands:

![alt text]({{ site.baseurl }}/images/Building-ffmpeg-form-nwjs-or-chromium-repository/image005.png "{{ page.title }}")


- **Force nwjs(chromium) to regenerate ninja files** for modified  *.h, *.asm, *.gypi (see previous step): 
  Go to nwjs source code location :  /node-webkit/src

~~~~ bash
src>gclientrunhook–force
~~~~

![alt text]({{ site.baseurl }}/images/Building-ffmpeg-form-nwjs-or-chromium-repository/image006.png "{{ page.title }}")


- Build ffmpeg code using ninja to generate ffmpegsumo.dll:

~~~~ bash
src>ninja -C out/Debug_x64 ffmpegsumo-j16
~~~~

![alt text]({{ site.baseurl }}/images/Building-ffmpeg-form-nwjs-or-chromium-repository/image007.png "{{ page.title }}")


- Build nwjs code using ninja and link to ffmpegsumo.dll:

~~~~ bash
src>ninja -C out/Debug_x64 nw -j16
~~~~

![alt text]({{ site.baseurl }}/images/Building-ffmpeg-form-nwjs-or-chromium-repository/image008.png "{{ page.title }}")