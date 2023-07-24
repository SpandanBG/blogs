# WebAssembly 

---

---

>The always-performant alter ego of Javascript.

## In the beginning there was Javascript

The language that browsers talked in. Initially it was just interpreted for purpose of speed of delivery. Soon browsers became competitive and then came in **JIT (Just In Time) Compiler**. The JIT Compiler optimised JS code that ran multiple times. The code would be opted-in or opted-out of optimised compilation based on the data-type it handled. If, say a method ran multiple times with the same data-type(s), it would be opted-in for optimisation. If, in the future the data-type changes, its behaviour would and thus it would be opted-out of optimisation. This gave performance, but inconsistent performance.

What if this performance could be made consistent? People thought Java could be the solution; but it failed for various reasons. Meanwhile there was something called the ASM.js which being developed and came out in 2013. It was a way to compile languages like C to JS as a low-level subset of Javascript. While it was bit more performant than regular JS, there was room for betterment. To reach that stage ASM.js needed to evolved.

## Then there was WebAssembly
Or **WASM** for short; the next step of ASM.js. It is** a portable binary format **that runs on browsers at near native speed. It is **not a substitute of Javascript but a compliment to it**. What it brings into the deck is the consistency of performance that Javascript struggles with.

This consistency of performance is brought in by the fact that the code is optimised and compiled before distribution on the web. Part of the optimisation is done based on the type of data that the program is suppose to handle. If it is known before hand, the compiler can optimise ways the data is accessed and operated on. As such, WASM modules are build by writing in any **statically typed language** that support WASM compilation. Amongst these language the notable ones are C/C++, Rust and AssemblyScript.

Does this mean that** WebAssembly is faster than Javascript? Not true**. Javascript can run as fast as WebAssembly, BUT it is inconsistent with performance. The consistency in performance of WASM brings in a huge change in how much and how quickly browsers can handle large computations. Currently WASM rans at 80% of native execution speed and will get better with future releases.

WASM support is currently available in major four browsers: *Chrome*, *Firefox*, *Safari* and *Edge*.

## Any to WASM

Well not any, but languages like C/C++/Rust to WASM. How a WASM module is build, is by leveraging the existing compilers for these languages. Before a high-level language is converted to the target machine language, it first gets converted to an intermediate representation (IR). WASM builders leverages on this IR to create a distributable **.wasm** file. As such, we get an optimised bytecode file ready to be distributed, quickly complied and executed. The notable compilers that allow you to make WASM files are:

- [Emscripten](https://github.com/emscripten-core/emscripten/wiki): A C/C++ compiler that was initially build to compile C/C++ to JS. Now it can build .wasm file.
- [wasm-pack](https://rustwasm.github.io/wasm-pack/): A Rust to WASM compiler that works along with the [wasm-bindgen](https://github.com/rustwasm/wasm-bindgen) library.
- [AssemblyScript](https://github.com/AssemblyScript/assemblyscript): A Typescript to WASM compiler that uses [Binaryen](https://github.com/WebAssembly/binaryen) for compilation to WASM.

A **.wasm** file created, are in binary format. This format can be transformed to text format for human human readability which is known as WebAssembly Text Format or a **.wat** file.

**The WebAssembly Binary Toolkit** ([WABT](https://github.com/WebAssembly/wabt)) gives you the tools play around with WASM; such as the `wasm2wat` which does the WASM to WAT conversion.


## WASM Execution

Currently there is no way to embed WASM files to your HTML files. It has to be fetched from your file delivery services to be complied and executed using the `WebAssembly` api given by the supported browsers. There are two ways to do the fetching and compilation:

- **Streaming Compilation**: As the name suggest, the .wasm file is complied as it is being fetched. This is done with the `WebAssembly.instantiateStreaming()`. An example code:

```js
WebAssembly.instantiateStreaming(fetch("doesSomething.wasm"))
    .then(wasm => {
      // play with wasm instance...
    })
    .catch(err => console.error(`--WebAssembly Error: ${err}`));
```

- **Non-Streaming Compilation**: AKA - the regular way.  Here the .wasm file is fetched, pushed as array buffer and then instantiated with `WebAssembly.instantiate()`. An example code:

```js
fetch("doesSomething.wasm")
    .then(res => res.arrayBuffer())
    .then(bytes => WebAssembly.instantiate(bytes)
    .then(wasm => {
        // play with wasm...
    })
```

## Basic Anatomy

As basic anatomy of a WASM module, it has the ability to **import functions** from Javascript and **export functions** to Javascript. As an example, we will write a simple WASM application that imports a function form JS and export a function that executes it with `42` passed in the parameter. We will write it in the WebAssembly Text Format and call it `answer.wat`:

```asm
(module
    (func $i (import "answer" "setAnswer") (param i32))
    (func (export "fortyTwo") (result i32)
        i32.const 42
        call $i
    )
)
```

*At line number 2* → We are importing a function named as **setAnswer** from the **answer** object - `(import "answer" "setAnswer")`; state that it would take the a parameter of integer of 32 bits - `(param i32)`; and it shall be referred as **$i** - `(func $i ... )`.

*At line number 3* → We are defining a function that returns an **integer of 32 bits** - `(result i32)` and export the function with the name **fortyTwo** - `(export "fortyTwo")`.

*At line number 4* → We are creating **an integer constant of 32 bit** with value 42 - `(i32.const 42)`.

*At line number 5* → We then  call the imported function with the reference name we gave - `call $i`.

WASM runs on a **stack based virtual machine** and has access to a **single linear memory.** This memory model is essentially an array of bytes that grows with the page size of 64kb. That means, with execution of each line of our exported function, it is going to push the data to the stack, and then, when the imported function is called, it is going to pop out the values from the stack and passed to the called function.

The `answer.wat` file can be complied to `answer.wasm` file with the help of **WABT**'s `wat2wasm` tool as such:

```shell
wat2wasm answer.wat -o answer.wasm
```

First let us create the **answer** object that has the function **setAnswer**. The setAnswer essentially would set the value of an DOM element.

```js
const answerObject = {
    answer: {
        setAnswer: ans => 
            document.getElementById("ans").innerHTML = ans;
    }
}
```

Now we shall write the JS code to fetch, compile and execute our "answering application":

```js
WebAssembly.instantiateStreaming(fetch("answer.wasm"), answerObject)
    .then(wasm => wasm.instance.exports.fortyTwo())
    .catch(err => console.error(`--WebAssembly Error: ${err}`);
```

*At line number 1* → We fetch our WASM file, compile it and pass the **answerObject** that we created.

*At line number 2* → We call the **fortyTwo** function that we exported from the WASM instance that we created.

As a result the DOM element with ID "*ans*" has 42 written on it.

## The Playground

As mentioned earlier, WASM module runs on a **Stack Based Virtual Machine** and currently owns a single linear memory. The linear memory is essentially an array of bytes with page size of 64kb. This memory can also be imported-to or exported-from the WASM module.

For importing, the memory has to be create using the `WebAssembly.Memory()` API in JS and passed to the WASM module. This means, multiple WASM modules can share the same memory instance amongst themselves, which gives a **dynamic linking** between between different modules.

It's also possible for functions in WASM modules can return pointers to the data in its linear memory to allow JS for direct access. While JS can access the linear memory of the WASM, WASM doesn't have direct access to the memory of JS. As such, if WASM module returns data instead it gets copied to the memory of JS.

Another API available is the the `WebAssembly.Table()` api. A table in a WASM module is a resizable typed array for holding references that can be accessed by both WASM and JS. References are kept separately from memory for safety, portability and stability purpose, since these values are not supposed to be read or written directly.

It is also possible to create globals with the help of `WebAssembly.Global()`. A global can be shared between JS and multiple modules of WASM.

## Piece them together

Web APIs in WASM are also currently not available in the current version. But they can be accessed using something called "glue code". These glue code are basically Javascript that can be called from the WASM module to invoke the Web APIs.

Other things WASM isn't able to do in the current release are: handle errors, understand datatypes other than integers (i32, i64) and floats (f32, f64). All these can be handled using glue code.

## Looking into the future

The future of WebAssembly seem bright. There are a lot more coming into the deck for play, like: threading (which currently you can achieve with WebWorkers), SIMD, exception handling, etc.

You can have a look at the upcoming features in WASM [here](https://webassembly.org/docs/future-features/).

## Appreciating the present

Even in the current state of WASM, a lot can be accomplished with it. Have a look as some of the things people have done with it:

- [Bar Code Scanner - eBay](https://www.ebayinc.com/stories/blogs/tech/webassembly-at-ebay-a-real-world-use-case/): eBay has used WASM to build a bar code scanner into their non-native app.
- [Squoosh](https://squoosh.app/): Squoosh app is an online image compression application made by Google.
- [Game Development](https://www.webassemblygames.com/): Game engines like Unity have now given support for WebAssembly exports.

With WebAssembly not only can you build your own complex applications, but also port in applications that were not even built for the web initially. All it takes it building wrappers over existing libraries.

## Motivating you

If you would like to know more about WebAssembly, you can start by following the references listed:

- [https://webassembly.org/](https://webassembly.org/)
- [https://developer.mozilla.org/en-US/docs/WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly)
- [https://rustwasm.github.io/docs/book/introduction.html](https://rustwasm.github.io/docs/book/introduction.html)
- [https://www.toomanybees.com/storytime/gluing-the-web-and-webassembly-together](https://www.toomanybees.com/storytime/gluing-the-web-and-webassembly-together)
- [https://blog.sessionstack.com/how-javascript-works-a-comparison-with-webassembly-why-in-certain-cases-its-better-to-use-it-d80945172d79](https://blog.sessionstack.com/how-javascript-works-a-comparison-with-webassembly-why-in-certain-cases-its-better-to-use-it-d80945172d79)
- [https://www.youtube.com/watch?v=njt-Qzw0mVY](https://www.youtube.com/watch?v=njt-Qzw0mVY)

---

Spandan Buragohain,
2019-05-29 10:12:27
