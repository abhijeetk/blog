---
layout: post
title: Up and running WebChimera with nwjs
---

- ### Add proxy to windows environment ###

~~~~ bash
set http_proxy="http://<user>:<password>@<ip>:<port>"
set https_proxy="http://<user>:<password>@<ip>:<port>"
~~~~

- ### Install build dependencies for WebChimera ###

Refer link : [https://github.com/RSATom/WebChimera.js#windows](https://github.com/RSATom/WebChimera.js#windows)
 
1. Visual Studio Community 2013/2015 - (I tried with VS2015)
2. Install VLC Player 32 bit and add it to PATH environment variable (Used to build webchimera)
3. CMake - cmake-3.6.0-rc2-win64-x64.msi
4. Node.js
5. Install **NW.js ia32** add it to PATH environment variable (Used to run player)

- ### Add proxy to npm packaging system ###

~~~~ bash
npm config set proxy http://<user>:<password>@<ip>:<port> 
npm config set https-proxy http://<user>:<password>@<ip>:<port>
~~~~

Check npm configuration in **C:\Users\<user>\.npmrc**

~~~~ bash
https-proxy=http://<user>:<password>@43.88.64.10:8080/
proxy=http://<user>:<password>@43.88.64.10:8080/
~~~~

- ### Check cmake generators ###

~~~~ bash
cmake --help
~~~~

![alt text]({{ site.baseurl }}/images/Up-and-running-WebChimera-with-nwjs-win/cmake.png "{{ page.title }}")

- ### Clone code and build it ### 

~~~~ bash
git clone https://github.com/RSATom/wcjs-ugly-demo.git
cd wcjs-ugly-demo/
build_nwjs.cmd
~~~~

- ### Run ###

~~~~diff
run_nwjs.cmd
~~~~

![alt text]({{ site.baseurl }}/images/Up-and-running-WebChimera-with-nwjs-win/nw-wcjs-win.png "{{ page.title }}")
