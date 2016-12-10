---
layout: post
title: Up and running WebChimera with nwjs Ubuntu
---

- ### Add proxy to ~/.bashrc ###

~~~~ bash
export http_proxy="http://<user>:<password>@<ip>:<port>"
export https_proxy="http://<user>:<password>@<ip>:<port>"
~~~~

- ### Add proxy to packaging system ###

If **sudo apt-get update** is not working, add below line to ***/etc/apt/apt.conf***.

Create /etc/apt/apt.conf if it doesn't exist and add line below

~~~~ bash
Acquire::http::Proxy "http://<user>:<password>@43.88.64.10:8080";
~~~~ 

~~~~ bash
sudo gedit /etc/apt/apt.conf
~~~~ 

- ### Install buil dependencies ###

~~~~ bash
sudo apt-get update
sudo apt-get install build-essential cmake libvlc-dev
sudo apt-get install vlc
sudo apt-get install git
sudo apt-get install npm 
npm -v
npm config set proxy "http://<userid>:<passwd>@43.88.64.10:8080"
npm config set https-proxy "http://<userid>:<passwd>@43.88.64.10:8080"
sudo npm install -g npm@latest
npm -v
~~~~

- ### Clone code and build it ### 

~~~~ bash
mkdir work
cd work/
git clone https://github.com/RSATom/wcjs-ugly-demo.git
cd wcjs-ugly-demo/
./build_nwjs.sh 
~~~~

- ### Run ###

~~~~diff
wget http://dl.nwjs.io/v0.12.0/nwjs-v0.12.0-linux-x64.tar.gz
tar -xvzf nwjs-v0.12.0-linux-x64.tar.gz 
export PATH=/home/abhijeet/work/wcjs-ugly-demo/nwjs-v0.12.0-linux-x64:"$PATH"
gedit index.html (Edit it to add http or https URL for video)
./run_nwjs.sh
~~~~

![alt text]({{ site.baseurl }}/images/Up-and-running-WebChimera-with-nwjs-ubuntu/WebChimera.png "{{ page.title }}")
