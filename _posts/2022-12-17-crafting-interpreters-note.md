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

#### Ch 2. Challenge Questions
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
* *Data types*: Built-in, "Atoms"
  * Booleans
  * Numbers: double-precision floating point numbers
  * Strings
  * Nil: Controversial but difficult to omit in a dynamically typed language
* *Expressions*: Produces a value, "Molecules"
  * Arithmetic 
    * *Operators*: (`+, -, *, /`)
    * *Operands*: subexpressions around operators
      * *Binary operators* since there are 2 of them e.g. 1+2
    * *Infix operators*: Fixed *in* between operands vs. *prefix* (before) or *postfix* (after)
    * "-" is both infix and prefix
  * Comparison and equality
    * Order: `<, >, <=, >=`
    * Equality: `==, !=`
  * Logical operators
    * Prefix operator: `!`
    * Control flow: `and`, `or`
      * *short-circuit*: They do not evaluate the second operand if they don't need to
  * Precedence and grouping
    * Similar to C's
* *Statement*: Produces an effect e.g. modifying state, reading input, producing output
  * Making `print` a built in allows us to start producing output earlier o
  * `expression statement`: expression followed by a `;`
  * `block`: Wrap of multiple statements by `{}` to produce one statment
* *Variable*:
  * Default is *nil*
  * `var v = "val"` or `var v`
* *Control Flow*: Conditionally run certain portions of code
  * *if*, *while*
* *Functions*:
  * *`aFunction(var1, var2)`*
  * Definition: *`fun printX(a) {...}`*
  * *Parameter* vs *Argument*:
    * *Parameter* (formal parameter, formals): Variable that holds argument value. *Declaration* has a *parameter* list. 
    * *Argument* (actual parameter): Actual value passed to function. Function *call* has *argument* list. 
  * *Declaring* vs *Defining* functions:
    * *Declaration*: Binds a function's type to its name for type checking. No function body.
    * *Definition*: Declares function and provides body for compilation.
    * Doesn't matter for dynamically typed language
  * Function body is always a *block*
  * *return* Functions return *nil* by default. You can explicitly return something else.
* *Closures*
  * *first class functions*: Real values you can reference, pass around and store in variables
  * *local functions*: Functions defined within functions
  * *Closures*: Local functions that hold unto surrounding local variables. It is necessary to maintain local variables used by local functions. You need a data structure that "closes over" and holds the variables needed.
    * Misnomer when used to describe *first-class function* even if function doesn't hold local variables
* Why Object-Oriented Programming (OOP)?
  * Useful for defining complex data types
  * Can scope methods to those classes without specifying the type in the name e.g. `hash-copy` for hashes in Racket.
* *Objects*
  * *Classes*: C++, Java
    * *Instances*: Store state of the object, hold reference to the class
    * *Classes*: Contain the methods and inheritance chain.
      * `Instance -(has class)-> Class -(inherits from)-> Class -> Method`
    * *Static dispatch*: Method lookup done at compile time using the known(?), static type of the instance
    * *Dynamic dispatch*: Looks up class of the instance at runtime
  * *Prototypes*: JavaScript
    * Only objects, no classes
    * Each object contains states and methods and may "delegate to" other objects
      * `Object -(delegates to)-> Object -(delegates to)-> Object`
#### *Classes in Lox*
  * Classes are first class in Lox
  * They can be declared and passed around
  * When the declaration is executed, the object is created and stroed in a variable
* Instantiation and initialization
  * Lox allows adding properties to objects
  * Use `this` to access a field/method on the current object in a method
  * `init` is called on object construction
* Inheritance
  * `class Brunch < Breakfast`
  * Brunch is a *derived class/subclass*
  * Breakfast is a *base class/superclass*
  * Use `super` to call in base class' method on our instance.
* Lox is not a true OOP language
  * Values of primitives are not real objects i.e. being instances of classes, they don't have methods or properties
  * TODO: A real language for users should fix this
#### Standard Library
  * Core implemented in the interpreter and for users to build on.
  * For Lox, just `clock()`
  * TODO: A real language should include:
    * String manipulation
    * Trig functions
    * File I/O
    * Networking
    * Reading input from user

#### Ch. 3 Challenge Questions
1. Sample Lox program. Edge case behaviors. Does it behave as expected?
  * Usage: `./jlox ch3.lox`
  * Edge cases:
    * Not including the `var` keyword should have thrown an error
    * Trying to return 

2. Open questions about language syntax and semantics. Suggested answers.
   * What happens when you have a property and method having the same name?
     * Do not allow it. Throw an error.
   * Are lambda expressions supported?
     * No
   * What is the behavior when a method is called on `nil`?
     * Throw a null pointer exception
   * Can the arithmetric operations be overloaded?
     * No
   * Do we have access to pointers? 
     * No 

3. Additional critical features apart from standard library
  * Formatting text
  * Cryptography
  * Randomness
  * Graphics
  * Math e.g. min, max, log
  * Timezones
  * Standard input and output