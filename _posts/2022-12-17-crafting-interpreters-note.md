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

#### Ch. 4 Challenge Questions
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
   * Are multi-line strings allowed?
   * Are escaped characters supported?

3. Additional critical features apart from standard library
  * Formatting text
  * Cryptography
  * Randomness
  * Graphics
  * Math e.g. min, max, log
  * Timezones
  * Standard input and output

### Design Note: Expressions and Statements

* Not all languages have statements usually functional languages e.g. most Lisps, Haskell
* Declarations and control flow constructs are expressions
* Easy:
  * `if` evaluates to the result of the chosen branch
  * variable declaration evaluates to the value of the variable
  * block evaluates to the last expresion in the sequence
* Trickier
  * What should a loop evaluate to?
    * In CoffeeScript, `while` evaluates to an array with each element the body evaluated to
* *Implicit returns* Functions return whatever the body evaluates to without need for explicit `return`. Common among languages that don't use.
  * Languages that *do* have statements allow a *=>* to specify this implicit return behavior,

## Ch 4. Scanning

* *REPL*: An interactive prompt. Read-Eval-Print Loop. From Lisp.
* It is good engineering practice to separate code that generates errors from code that reports the error
  * Examples: on stderr, error window, logged to a file. All these should be handled in one place
  * Errors will be generated in multiple locations and they should not be the ones to report the errors
  * An `ErrorReporter` interface passed to the scanner, parser might be the best option
* *Lexeme*: Smallest sequence of characters that mean something in the language
  * e.g. `var` `language` `=` `"lox"` `;`
* *Token*: Lexeme + Metadata:
  * *Token type*: What kind of lexeme is represented e.g. left parenthesis, semicolon, string etc.
* *Literal*: Numbers or strings
* *Location information*: Sophisticated error handling implementations should include the column, length, and line the error occured.
  * Some implementations store location with 2 numbers:
    * Offset from beginning of source file
    * Length of lexeme
  * Convert offest to line:
    * Count the number of preceding newlines
    * Slow but only calculated when needing to display line and column info
    * Since most tokens are not shown in an any error message, it's worth avoiding this computation

### Regular Languages and Expressisions
* Scanner loops and progressively finds a lexeme and emits a token
  * A *regex* may be used to match the lexeme
* *Lexical grammar*: Rules that describe how a language groups characters into lexemes
* *Regular languages*: Rules that are simple enough where grammar can be defined with lexical grammar (?)

#### Recognizing Lexemes
* It's good UX to report as many errors as possible
* *Lookahead*: Used in the scanner to get the next character without advancing the current scanning pointer. The smaller the lookahead, the faster the scanner.
* *Maximal munch*: When two rules can match a chunk of code, the one that matches more should be chosen. If we can match `orchid` or `or`, we go with `orchid`. e.g. in C, `---a;` is scanned as `-- -a` instead of `- --a`.
* *Reserved word*: identifier reserved by the language.

#### Design Note
* When designing a new language, avoid explicit statement terminators e.g. `;`

#### Ch. 4 Challenge Questions
1. Python and Haskell grammar are not *regular* It means the lexemes cannot be lexed with regular expressions. Python and Haskell are indentation-sensitive. Lexers need to capture the change in indent/unindent and it's not possible with regular expressions [Source](https://www.reddit.com/r/compsci/comments/kkzn3r/the_lexical_grammars_of_python_and_haskell_are/).

## Ch. 5: Representing Code

* We want a code representation that's:
  * Simple for the parser to create
  * Easy for the interpreter to run

* Idea: **Tree** of operations that we act on using *post-order traversal.*

### Context-Free Grammers (CFG)
* Grammer for lexemes (*regular languages*) is not expressive enough for expressions (e.g. arbitrary nesting)

| Terminology | Lexical grammer | Syntactic grammer |
| ----------- | --------------- | ----------------- |
| Alphabet    | Characters      | Tokens            |
| Words       | Lexeme/Tokens   | Expressions       |
| Implementer | Parser          | Interpreter       |

* Key idea: Define *rules* to generate infinite valid strings
  * *Derivations:* Strings *dervied* from the rules
  * *Productions:* Rules used to *produce* the strings
  * *Head:* Name of the CFG
  * *Body:* Describes what the CFG creates 
  * *Terminal:* A 'letter' in the grammar. Does not lead to new strings.
  * *Nonterminal:* Named reference to another rule.
  * *Backus-Naur Form:* Way to specify grammar
    * breakfast -> protein `with` breakfast `on the side`;
    * protein -> crispiness crispy `sausage`
    * crispiness -> `really`
    * crispiness -> `really` crispy
    * protein -> `eggs`
* Recursions where there are *productions* on either side of a *terminal* are not regular
  * Why? We need to track the number of terms to match either side which is not possible with regular grammars
* Improving CFG notation:
  * breakfast -> protein (`with` breakfast `on the side`)?
               | bread ;
  * protein   -> `really`+ `crispy` `bacPon`
               | (`poached` | `fried`) `eggs` ;
  * bread     -> `toast` | `buscuits` ;
* CFGs help crystallize the informal syntax

### Grammar for Lox Expressions

* The syntactic grammar, different from the lexical grammar, is much larger. 
* Beginning grammar is (currently ambiguous):
  * ```sh
    expression -> literal | unary | binary | grouping;
    literal -> NUMBER | STRING | "true" | "false" | "nil";
    grouping -> "(" expression ")"
    unary -> ("!" | "-") expression;
    binary -> expression operator expression;
    operator -> "==" | "!=" | "<" | "<=" | ">" |
                ">=" | "+"  | "-" | "*"  | "/";
    ```

### Syntax Trees
* We will use a data structure, *syntax tree* that captures our syntax
  * It's going to be a *tree* since since some *heads* e.g. `unary` refer back to an `expression`. 
  * *Abstract Syntax Tree (AST)* skips (elides) productions that are not needed
  * In a *Parse Tree*, every production becomes a node in the tree
 
### Disoriented objects
* Should the `Expression` class include implementation logic?
  * Probably not, as they're owned by other classes
  * These classes are a way to communicate between the *parser* and *interpreter*

## Working with Trees
* Having `if-else` blocks for handling different types of `expressions`s is suboptimal
  * `Expression`s later in the blocks will run slower
  * We could use attach an `interpret()` method on each `Expression` (*Interpreter pattern*)
    * This doesn't scale well as the class is used in multiple domains
    * Object-oriented and functional languages have pros and cons in ease of ('expression problem'):
      1. Adding a new class e.g. `BinaryExpr`
      2. Adding a new method e.g. `interpret()`
 * *The Expression Problem:* How do you add new classes and operations on the classes.
   * Object-Oriented languages make it easier to add new classes () 
   * Functional languages make it easy to add new functions (pattern matching)
   * Neither style makes it easy to add both classes and functions
 * *Visitor Pattern:* Approximate functional style in OOP by adding a layer of indirection.
   * Consider:
     * `Pastry` with subclasses `Beignet, Cruller`
     * Subclasses define `accept(PastryVisitor visitor)`
     * `PastryVisitor` contains `visitBeignet, visitCruller` etc.
     * Subclasses implement their specific visitor

## Ch. 6: Parsing Expressions

* 