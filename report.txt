"Oh": A Report on the Stack-Based Mind Playground

1.0 Introduction: Philosophy and Purpose

The "oh" programming language is more than a simple tool; it is a personal project that reflects its creator's evolving understanding of computer science and language design. It serves as both a mirror of existing knowledge and a crucible for forging new ideas. This section establishes the core identity of the language, its naming convention, and its primary design goals, providing a philosophical foundation before exploring its mechanics.

The name "oh" was chosen for its simplicity and personal significance to the author, who uses "oh" and "meh" as personal equivalents to the common placeholder names "foo" and "bar." This pragmatic choice underscores the language's nature as an intimate, personal creation rather than a formal, committee-designed system.

At its core, "oh" is a Reverse Polish Notation (RPN) stack-based language. It is written entirely in JavaScript and is engineered for deep, seamless interoperation with its host environment. This design choice is not accidental but is central to its purpose.

The language's primary goals are:

* JavaScript Interoperation: To inherit and provide full access to all browser APIs and the broader JavaScript ecosystem, allowing it to perform any task that JavaScript can, albeit in its own distinct style.
* Personal Playground: To function as a frictionless environment for its creator to experiment with programming concepts. It is a space to test, integrate, evolve, or discard new ideas with minimal overhead.
* A Tool for Understanding: To act as both a reflection of and a tool to amplify the creator's comprehension of programming. Its evolutionary nature is a key feature, with the author noting that the entire interpreter is periodically rewritten from scratch as their understanding outgrows the existing implementation.

This philosophy is realized through a simple yet powerful two-phase interpreter loop, which forms the mechanical heartbeat of the language.

2.0 Core Mechanics: The Interpreter's Heartbeat

To understand "oh," one must first understand the strategic design of its interpreter's core loop. Unlike many interpreted languages that read and execute code in a single pass, "oh" employs a distinct two-phase process: a "compile time" phase where code is transformed into a collection of functions, and a "runtime" phase where those functions are executed. This separation is central to the language's philosophy of delaying execution and pre-computing as many decisions as possible.

The interpreter's basic unit of work is a token. A token is a string of characters read from the source code, typically separated by spaces, newlines, or tabs. Certain special characters are treated as tokens in their own right, even without surrounding whitespace. These are: (, ), [, ], {, },   , and "`.

The interpreter processes these tokens through two primary phases:

* Compile Time: This is the process of reading tokens from the source and transforming each one into a JavaScript function. These functions are then collected in an array. The author clarifies that this is not true compilation in the traditional sense, but rather a method of "delaying executions and precomputing decisions" to resolve words and syntax before the code is ever run.
* Runtime: This phase occurs after the entire source has been processed. The interpreter simply iterates through the array of functions generated during compile time and executes each one sequentially.

While pre-resolving words at compile time offers a performance benefit by avoiding repeated lookups, the author notes that this is not the primary motivation for this design. The critical insight here is that the main reason for the compile phase is to achieve a form of static binding. By finding a word's definition and capturing a reference to its function at compile time, that reference becomes "frozen." This ensures that if the word is later redefined, the already-compiled code remains unaffected, thus preventing the complexities of dynamic binding.

This entire process is encapsulated in the Read-Eval-Print-Loop (REPL). To illustrate, consider the input 1 2 3:

1. Read: The REPL reads the entire line of source code.
2. Tokenize: It repeatedly calls an internal function, read_word(), to get each token from the source: "1", then "2", then "3".
3. Compile Element: Each token is passed to a core function named compile_element. This function attempts to find a matching "word" definition in the current environment.
4. Compile Atom: When compile_element fails to find a defined word for "1", "2", or "3", it falls back to the compile_atom function, which attempts to parse the token as a literal or special syntax. In this case, compile_atom matches the tokens against a regular expression for numbers. For each match, it generates and returns a JavaScript closure that, when executed, will push the corresponding number onto the stack (e.g., () => stack.push(1)).
5. Collect and Execute: The REPL collects these three generated functions into an array: [() => stack.push(1), () => stack.push(2), () => stack.push(3)]. Once the input is exhausted, the REPL enters the runtime phase, iterating through this array and executing each function in order. The result is that the numbers 1, 2, and 3 are pushed onto the stack.

This fundamental process of converting every token into a delayed function is the foundation upon which all other language features, including user-defined words and complex control structures, are built.

3.0 Building Blocks: Words, Definitions, and Environments

The expressive power of "oh" emerges from its ability to define new "words," which extend the language's vocabulary and functionality. These definitions are created and stored within structures called environments, which provide a form of lexical scoping that is resolved entirely at compile time. This section breaks down how new words are created, how environments manage their scope, and how code can be organized into reusable modules.

Defining Words with :

The primary interface for creating new definitions is the colon (:) word. The colon is an immediate word, which means it executes during the compile time phase rather than being collected as a function for the runtime phase. Its job is to build a new word and add it to the current environment.

Consider the following example:

: push-three-numbers 1 2 3 ;


When the compiler encounters :, it executes immediately and performs the following steps:

1. It reads the next token, push-three-numbers, as the name for the new word.
2. It then begins compiling all subsequent words (1, 2, 3) until it encounters the semicolon (;) delimiter.
3. Each of these words is compiled into a function and collected into an internal array, just as the REPL does.
4. Finally, it creates a new function that will execute the collected functions in sequence and assigns this new executable function to the name push-three-numbers in the current environment.

Understanding Environments

An environment in "oh" is a simple JavaScript object containing two key properties: a word lookup table (an object that maps names to functions) and a parent pointer to another environment. This structure creates a chain of environments, which the interpreter searches when looking up a word. A global env pointer tracks the current environment during compilation.

This system enables compile-time lexical scope. This lifecycle—creation at the start of a definition, active use during compilation, and automatic disposal upon completion—is the cornerstone of "oh's" scoping. When a word like : begins a definition, it first generates a new, temporary child environment. This new environment becomes the active env pointer for the duration of the definition's body. Once the definition is complete, the env pointer is restored to its parent, and the temporary environment is discarded.

However, any words looked up during this period are "frozen" as function references in the new word's compiled code. For example:

: outer-definition
  : inner-definition 1 2 3 ;  -- Created in a temporary env, local to outer-definition's compilation.
  inner-definition             -- The function for inner-definition is found and captured here.
;


When outer-definition is compiled, inner-definition is created within a temporary environment specific to outer-definition. When inner-definition is then used, its compiled function is captured and placed in the code array for outer-definition. Even though the temporary environment is later destroyed, the inner-definition function remains embedded within outer-definition's definition.

Modules and Imports

To organize and reuse collections of definitions, "oh" provides a module system.

* The module word creates a named, reusable environment. It compiles a block of code and creates a new immediate word that, when executed at compile time, pushes the module's environment onto the stack.
* The import and import-all words are immediate words that consume an environment from the stack. At compile time, they take this environment and inject either specified words (import) or all of its words (import-all) into the current compile-time environment, making them available for use.

This mechanism allows developers to create libraries of functions and selectively include them in new definitions, maintaining clean separation and promoting code reuse. From these static definitions, the language builds further to incorporate dynamic, stateful features.

4.0 Managing State: Bindings and Mutability

While the stack is the primary mechanism for data flow in "oh," managing complex state solely through stack manipulation can become cumbersome. To address this, the language introduces "bindings," which act as named, variable-like constructs that help reduce stack juggling and provide a more intuitive way to handle state.

Creating Bindings with bind and declare

The primary word for creating a stateful binding is bind. It is classified as an immediate 1 word, a special type of immediate word that executes at compile time but is also required to return a function for the final runtime execution array.

The bind word performs a distinct two-part action:

1. At Compile Time: It reads a name from the source code and creates a "binding." A binding is a JavaScript function that also acts as a data cell (e.g., const binding = () => put(binding.value)). This function, when called, pushes its own internal value onto the stack. The binding is immediately registered under the given name in the current compile-time environment.
2. For Runtime: bind then creates and returns a "setter" function (e.g., () => binding.value = stack.pop()). This function is added to the execution array. When the code is finally run, this setter will pop a value from the stack and assign it to the binding's internal state.

The critical distinction between bind and declare lies in this runtime component. declare is an immediate 0 word. It also creates a binding at compile time, but it simply initializes the binding's internal value to 0. It does not return a setter for the runtime array, as it doesn't need to consume a value from the stack.

The Nature of Bindings: Dynamic yet Scoped

Bindings in "oh" are fundamentally dynamic. Because a binding is a reference to a single function object, any part of the code that holds a reference to it can mutate its value using a mutator word. This change is immediately reflected everywhere that binding is used, as subsequent calls will push the new value.

Despite this dynamic behavior, the compile-time lexical scoping rules still apply. If an inner definition creates a new binding with the same name as one in an outer scope, it effectively shadows the outer binding. Any use of that name within the inner scope will refer to the new, local binding for the duration of its compilation. The outer binding remains unaffected and becomes visible again once compilation leaves the inner scope.

Mutability and JavaScript Interop

To facilitate state changes, "oh" provides a unified mutator interface. This is a factory function in the interpreter that generates immediate words like set, increment, and decrement. At compile time, a mutator word like set finds a target by name (a binding, a delayed binding, or a JS property) and returns a specialized function for the runtime array. At runtime, this function executes the specific mutation logic—for set, this involves popping a value from the stack and assigning it to the pre-resolved target.

For direct interaction with JavaScript objects, "oh" provides "dot notation" syntax sugar:

* Getters:
  * To get a property from a named object, use a token like body.style.color. The interpreter compiles body as a word and then generates code to access the subsequent property chain from the object it pushes.
  * To get a property from an object already on the stack, use a token that starts with a dot, like .style.color.
* Setters:
  * Setters are formed by appending an exclamation mark (!) to a getter token, such as 'red' body.style.color!.
  * Alternatively, the set mutator can be used: 'red' set body.style.color or 'red' body set .style.color.

A convenience feature automatically translates kebab-case property names in dot notation (e.g., text-content) to the camelCase (textContent) required by JavaScript, simplifying DOM manipulation.

This system of compile-time resolution and runtime mutation works effectively for many use cases, but it encounters challenges when the purely compile-time model must interact with runtime realities, leading to a frontier of more experimental solutions.

5.0 The Experimental Frontier: Runtime Environments and Asynchronicity

This section explores the evolving features of "oh" that push the boundaries of its original design. The language's initial goal was to resolve all lookups and decisions at compile time to create a simple, efficient runtime. However, practical needs, particularly for interactive browser applications, have challenged this philosophy, leading to the introduction of new, experimental concepts that the author candidly describes as "hacks."

The Need for Runtime State

A core conflict arises from the language's design: the interpreter was built to resolve everything at compile time, but features like the dom word require environments that can exist and be modified at runtime. To bridge this gap, "oh" introduces several mechanisms that break the original pattern but enable crucial runtime flexibility.

* Delayed Lookups: The word: syntax (a word name followed by a colon) defers the search for a word from compile time to runtime. When the compiler sees this, instead of looking up the word immediately, it generates a function that will perform the lookup when it is executed.
* Delayed Bindings: The :name syntax allows for the creation of new words in the current environment at runtime. When compiled, it generates a function that, at runtime, will pop a value from the stack and create a new word. The :name: variation does the same but also pushes the value back onto the stack immediately after creation. The generated function is conceptually similar to this:

Runtime Environments with block and defun

To manage runtime state, "oh" introduces the block and defun words.

* The block word is an immediate 1 word that compiles a body of code (delimited by end). It returns a single function that, when executed at runtime, performs three steps:
  1. Creates a new environment.
  2. Executes the pre-compiled code within that new environment.
  3. Restores the previous environment.

This mechanism, combined with delayed lookups, provides what the author calls a "fake lexical scope" at runtime. It's acknowledged as a "hack" because it relies on the coincidence of a delayed lookup executing while a temporary runtime environment is active.

* The defun word is a close relative of block, creating a reusable, named word that executes a block each time it is called.

Asynchronous Operations with wait

Another experimental feature is the "auto await mode," designed to handle asynchronous JavaScript operations like fetch.

* The wait word is an immediate word that overwrites the interpreter's core make_sub function (which builds the final executable from a list of compiled functions) with an async version.
* At runtime, this new async version executes each function in the sequence and then immediately checks if the top item on the stack is a Promise.
* If a Promise is found, the interpreter awaits its resolution. If the promise resolves with a value, that value is pushed back onto the stack before execution continues.

This allows for a clean, linear syntax for asynchronous code, as seen in the example 'url' fetch -text log, where the results of chained promises are handled automatically.

These experimental features are not viewed as design flaws but as part of a deliberate methodology. As the author states, "The only way this feature... will evolve is by pushing their current implementation, solution or hack to the limits and see how they explode to know the direction that must be taken." These intentional stress tests reveal the path forward for the language's next evolution, particularly toward a proper component abstraction for more robust UI development.

6.0 The "Oh" Language in Practice: Key Abstractions and Tooling

This final section covers some of the high-level abstractions and practical tools built into "oh." These features demonstrate how the language's core concepts—such as immediate words, list processing, and runtime execution—combine to create powerful, domain-specific functionality for tasks like DOM manipulation, conditional logic, and debugging.

The dom Abstraction

The dom word is a runtime function that provides a declarative, s-expression-based syntax for creating HTML DOM elements. It takes a nested list from the stack and recursively converts it into a tree of live DOM elements, which can then be appended to the document. The structure of the list uses special directives to configure the elements.

Directive	Example	Purpose
#id	#the-title	Sets the element's id attribute.
.class	.heading	Adds a class to the element's classList.
-property	-type text	Sets an attribute on the element.
@event	@click (...)	Attaches an event listener. This directive triggers a unique runtime compilation process: 1. A new, temporary environment is created. 2. A word named event (which pushes the JS event object to the stack) is injected into this new environment. 3. The provided list is then compiled into a handler function using this environment, capturing the event word.
:name	:my-button	Creates a word at runtime that pushes the element onto the stack.
reader	reader name prop	Creates a "getter" word for an element property.
writer	writer name prop	Creates a "setter" word for an element property.

Control Flow: if

Conditional logic is handled by the if word, which is an immediate 1 word. It provides a structure for branching based on a runtime value from the stack. The syntax is:

if [test-code] then [true-branch] else [false-branch] end

At compile time, if reads and compiles three separate blocks of code: the test, the true branch, and the optional false branch. It then returns a single function to the execution array. At runtime, this function executes the test block, pops the result from the stack, and uses it to decide whether to execute the true branch or the false branch.

Debugging with trace

The trace mechanism is a debugging tool implemented as a "hack" that hijacks the compilation process. When the trace word is executed, it replaces the standard compile_in_list function with a modified version. This new version wraps every subsequently compiled function in another function that, after execution, logs the current state of the stack to the console. This design has a significant advantage: the performance overhead of tracing is only incurred when it is explicitly enabled. The no-trace word restores the original, non-logging compiler function, allowing for detailed introspection without permanently slowing down the interpreter.


--------------------------------------------------------------------------------


In summary, "oh" stands as a deeply personal and continuously evolving exploration of language design. It prioritizes conceptual purity and rapid experimentation over production-ready stability, serving as a powerful, albeit challenging, tool for its creator. It is, in the author's own words, a true "playground for my mind."
