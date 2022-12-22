---
layout: post
title:  "Crafting Interpreters Notes"
date:   2022-12-17 21:00:00 -0500
categories: programming crafting-interpreters
---

Personal notes for new terms and concepts I come across in the [Crafting Interpreters](https://craftinginterpreters.com/) book.

## Ch. 1: Introduction

* *Domain-specific languages*:
  * Languages tailored for specific tasks. Examples:
    * scripting languages e.g. AWK, Bash
    * markup formats e.g. CFML
    * template engines e.g. Jinja
    * configuration files e.g. YAML 

## Ch. 2: A Map of the Territory

### Language's Implementation vs. Language
* Language: The language specification as experienced by the user
* Language implementation: Details on how a language's specifications are implemented e.g. using bytecode, stacks, recursive descent etc

### Part of a Language

* *Scanning* (lexing): Text (characters) to tokens (words)
* *Parsing*: Tokens to syntax tree (AST, nested tree of grammar)
  * Errors not conforming to language's grammar are *syntax errors*
* *Static analysis*: 
  * *Binding*: Wiring identifiers to where they're defined
  * *Type checks*: In static languages. Check that types are compatible.
  * Errors with wrong types are *type errors*
* *Intermediate representations (IR)*:
  * Source language e.g. C <-> IR <-> target architecture e.g. x86
* *Optimization*: IRs allow us swap user-provided semantics for more efficient semantics. But there might be some architecture-specific optimizations we might miss out on.
* *Code generation (code gen)*: Convert IR to code the CPU can run. Two options
  * *Generate CPU code*:
    * Fast but difficult and complex to implement for each architecture.
    * Compiler tied to one architecture
  * *Generate virtual machine code (bytecode)*:
    * Code for idealized machine
* Compiling bytecode
  * Mini-compilers (easier) to convert from bytecode to architecture code
  * Write a virtual machine (VM) that simulates CPU e.g. VM in C allows to support any architecture with a C compiler
* *Runtime*: All services the language provides while running the program e.g. garbage collection, "instance of" representations
  * Runtime implementation is inserted in executable for fully compiled programs
  * Copy of runtime is embedded in Go program
  * If language is run in interpreter of VM, runtime is embedded there
* *Single-pass compilers*: Compilers that interleave parsing, analysis, and code generation. No intermediate data structures and revisiting of parts already visited.  
  * Limit the design of the language
  * Examples: Pascal, C. It's why, in C, you can't call a function before defining it unless you have a forward declaration. 
* *Tree-walking interpreters*: Interpreters that traverse the AST after parsing the program.
  * Typically slow
* *Transpiler (source-to-source compiler, transcompiler)*: Converts the source programming language to another programming language
  * Writing backend requires a lot of work
  * Treat source of another language as the intermediate representation
  * Earlier on, around when UNIX became popular and C compilers came with UNIX, C was a good target language
  * Today, JS, the "machine code" of browsers, is a popular target language. Although WebAssembly, a lower language, is growing in popularity
* *Just-in-time compilation (JIT)*: 
  * Executing as machine code is the fastest way to execute a program
  * But we might not know the target architecture
  * When the program is loaded (as source or bytecode), it is compiled to the target architecture
  * The generated code is profiled (using profiling hooks) and recompiled and optimized to improve performance
    * Olu: This is quite interesting. Curious how it applies signals to improve performance. 
  * Used by HotSpot Java Virtual Machine (JVM), Microsoft's Common Language Runtime (CLR) and JavaScript interpreters.
  * Question: What if we're not able to compile to the architecture?
 
#### Compiler vs. Interpreter

* Not a binary choice e.g. fruit (botanical term) vs. vegetable (culinary term)
* *Compiling:* Converting source language to a (usually lower-level) form e.g. bytecode, machine code. Transpiling is also compiling. 
* *Compiler*: An implementation that takes source code and outputs an executable that needs to be run separately.
* *Interpreter*: An implementation that takes source code and executes it immediately.
* Compilers and Interpreters:
  * Compilers only: 
  * Interpreters only: *Matz’s canonical implementation of Ruby**. Parses and executes from source.
  * Compilers and interpreters: *CPython*. From user's POV, the python is run from source but CPython compiles source to bytecode and runs in a VM.

#### Challenge Questions
* *Dart* as open-source language:
  * Scanner and parser are hand-written (no `.l`, `.y`): [source](https://github.com/golang/go/blob/master/src/go).
* *JIT Downsides*:
  * There is overhead to analysis and compiling during runtime
  * Compilation might be wasted if analysis and recompilation happens towards the end of the program's run [stackoverflow](https://stackoverflow.com/questions/538056/jit-compiler-vs-offline-compilers)
* Why Lisp interpretations that compile to C also contain an interpreter that execute Lisp on the fly.
  * *I'm not sure if this is correct*
  * An interpreter allows for a faster development cycle since you can more easily run the code as you build [HN](https://news.ycombinator.com/item?id=16169267)
  * The code might be run in an environment with limited resources e.g. memory


## Ch. 3: Lox Language

* Lox is *dynamically typed*: Variables can hold any values
* Managing memory:
  * Reference counting
    * Easier to implement but significant limitations
  * Tracing garbage collection (garbage collection, GC)
    * Notoriously difficult to implement and debug
* Built-in data types: 
  * Booleans
  * Numbers: double-precision floating point numbers
  * Strings
  * Nil: Controversial but difficult to omit in a dynamically typed language