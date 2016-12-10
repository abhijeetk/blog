---
layout: post
title: Up and running ASM.js in browser
---

- ### Install Emscripten SDK ### 
Download Emscripten SDK Offline Installer and install it. For more information about installation process refer [https://kripken.github.io/emscripten-site/docs/getting_started/index.html](https://kripken.github.io/emscripten-site/docs/getting_started/index.html)

- ### Check Emscripten installation ###
Click on *Windows -> Search* and search for Emscripten Command Prompt. Check Emscripten version.

~~~~ bash
emcc -v
~~~~

**emcc is transcompiler to compile C/C++ code to javascript code.**

![alt text]({{ site.baseurl }}/images/Up-and-running-ASM-js/emcc_version.png "{{ page.title }}")


- ### Create C/C++ test file ###

~~~~ bash
mkdir asmdemo
cd asmdemo
copy NUL hello-world.c (create hello-world.c)
~~~~

Copy following content into newly created hello-world.c

~~~~ bash
#include<stdio.h>
int main() {
  printf("hello, world!\n");
  return 0;
}
~~~~

- ### Compile C/C++ code using emcc ###

~~~~ bash
emcc hello-world.c
~~~~

it will create file name a.out.js

- ### Run  ###

~~~~ bash
node a.out.js
~~~~

![alt text]({{ site.baseurl }}/images/Up-and-running-ASM-js/compile-n-run.png "{{ page.title }}")

- ### ASM.js in Browser ###
- We want HTML so that we can view it in the browser. Enter this command to specify and HTML document.

~~~~ bash
emcc hello-world.c -o hello.html
~~~~

- Open hello.html that with your browser, and you can see it as a web page.

![alt text]({{ site.baseurl }}/images/Up-and-running-ASM-js/emcc-in-browser.png "{{ page.title }}")
