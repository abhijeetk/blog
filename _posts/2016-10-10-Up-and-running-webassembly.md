---
layout: post
title: Up and running webassembly in browser 
---
## ASM to WebAssembly ##

- ### Prerequisite ###

1. Visual Studio 2015
2. cmake

- ### Clone binaryen(transcompiler for wasm) ###

~~~~ bash
mkdir wasmdemo
cd wasmdemo
git clone https://github.com/WebAssembly/binaryen.git
cd binaryen
~~~~

- ### Build code ###

~~~~ bash
wasmdemo\binaryen>cmake .
wasmdemo\binaryen>cmake --build . --config Release
~~~~

![alt text]({{ site.baseurl }}/images/Up-and-running-webassembly/binaryenBuild.png "{{ page.title }}")

- ### Utilities gets generated after build in binaryen/bin ###

![alt text]({{ site.baseurl }}/images/Up-and-running-webassembly/Utilities.png "{{ page.title }}")

- ### Create hello_world.asm.js file with content below ### 

~~~~ bash
function MyMathModule(global) {
    "use asm";
    var exp = global.Math.exp;
    function doubleExp(value) {
        value = +value;
        return +(+exp(+value) * 2.0);
    }
    return { doubleExp: doubleExp };
}
~~~~

![alt text]({{ site.baseurl }}/images/Up-and-running-webassembly/asm2wasm.png "{{ page.title }}")

- ### wasm-as.exe is a wasm assembler which converts intermediate AST to wasm binary ###

~~~~diff
wasmdemo\binaryen\bin>wasm-as.exe hello_world.wast -o hello_world.wasm
~~~~

![alt text]({{ site.baseurl }}/images/Up-and-running-webassembly/wasm-as.png "{{ page.title }}")

## Running WebAssembly in Browser ##

- We need to access WebAssembly on the Web is through an explicit JS API.
Refer [https://github.com/WebAssembly/design/blob/master/JS.md](https://github.com/WebAssembly/design/blob/master/JS.md) for more information.

- Create file *index.html* with content below :

~~~~ bash    
<script>
fetch("hello_world.wasm")
.then(function(response) {
return response.arrayBuffer();
})
.then(function(buffer) {
var dependencies = {
"global": {},
"env": {}
};
dependencies["global.Math"] = window.Math;
var moduleBufferView = new Uint8Array(buffer);
var myMathModule = Wasm.instantiateModule(moduleBufferView, dependencies);
console.log(myMathModule.exports.doubleExp);
for(var i = 0; i < 5; i++) {
console.log(myMathModule.exports.doubleExp(i));
}
});
</script>
~~~~

![alt text]({{ site.baseurl }}/images/Up-and-running-webassembly/server_content.png "{{ page.title }}")

- Run code through local server and open file in **Chrome Canary** web browser.

*Other browsers might give an error as binaryen version used to build wasm file and binaryen version inside browser is different.*

~~~~ bash
TypeError: wasm validation error at offset 8: failed to match binary version
~~~~

- ### Output ###

![alt text]({{ site.baseurl }}/images/Up-and-running-webassembly/output.png "{{ page.title }}")

- ### References ###

[http://cultureofdevelopment.com/blog/build-your-first-thing-with-web-assembly/](http://cultureofdevelopment.com/blog/build-your-first-thing-with-web-assembly/)
[https://github.com/WebAssembly/design/blob/master/JS.md](https://github.com/WebAssembly/design/blob/master/JS.md)
[https://github.com/brakmic/brakmic/blob/master/webassembly/COMPILING_WIN32.md](https://github.com/brakmic/brakmic/blob/master/webassembly/COMPILING_WIN32.md)
