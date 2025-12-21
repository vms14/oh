An Extensive Beginner's Tutorial for "oh": A Mind Playground

1.0 Introduction: Welcome to the "oh" Playground

Welcome to "oh," an experimental, personal, stack-based programming language written entirely in JavaScript. "oh" was not designed to be a production language but rather a "mind playground"—a fluid and minimalist environment for exploring the fundamental concepts of programming, from compilation and scope to syntax and interoperability. This document serves as a comprehensive tutorial, transforming the creator's raw, scattered notes into a structured guide for anyone curious about how such a language is built from the ground up.

At its heart, "oh" is a direct reflection of its creator's evolving understanding of programming. It is a tool for both expressing and amplifying that knowledge, as captured in the author's core philosophy:

In a way it is a reflection of my current understanding about programming. Everything i know about programming is being reflected either as part of the core interpreter or as a word definition. It grows when my understanding grows. When my understanding grows and the current implementation does not match my understanding anymore, it gets rewritten from the scratch. But it also amplifies my understanding. It is a playground for my mind and a training process to teach me about almost any concept about programming.

The language is guided by two primary goals: first, to achieve maximum interoperability with JavaScript, thereby inheriting the vast ecosystem of browser APIs. Second, to provide a platform where new ideas can be tested with minimal friction, allowing for rapid experimentation with syntax, control flow, and language design.

To understand how "oh" achieves this flexibility, we must first explore the fundamental mechanics that drive its interpreter.

2.0 The Core Philosophy: Compile Time vs. Runtime

The key to unlocking the "oh" language lies in understanding its two distinct operational phases: compile time and runtime. This strategic separation is not about compiling to machine code but about pre-computing decisions and delaying execution. This approach is central to how the language achieves both its unique flexibility and its surprising performance.

Compile Time in "oh" is the process of reading text tokens from the source code (e.g., the string "1", the word +) and converting them into an array of executable JavaScript functions. This phase is less about traditional compilation and more about "delaying executions and precomputing decisions." Every number, word, or piece of syntax is resolved into a function that, when called later, will perform the intended action.

Runtime is the subsequent phase, and it is elegantly simple. The interpreter takes the array of functions generated during compile time and iterates through it, executing each function in sequence. There are no complex lookups or decisions to be made at this stage; all of that work has already been done.

This compile-time-first approach provides two significant benefits that define the language's behavior:

* Performance: By performing a word lookup only once during compilation, the interpreter avoids significant runtime overhead. As the language's creator notes, "It is not the same to lookup what a word means once, than 200 times in a loop." In "oh", the meaning is resolved once, and the resulting function is simply executed repeatedly at runtime.
* Definition Freezing: This approach effectively avoids dynamic binding. When a word is compiled, its current definition (the JavaScript function it points to) is "frozen" into the code. If that word is later redefined, the already-compiled code remains unaffected because it holds a direct reference to the original function. This ensures predictable and stable behavior, as the meaning of code does not change unexpectedly at runtime.

The engine that drives this entire process—the mechanism responsible for turning simple text tokens into this powerful array of functions—is a single, core function called compile_element.

3.0 The Heart of the Interpreter: compile_element

The compile_element function is the absolute core of the "oh" interpreter. Its design is governed by a single, unifying principle: to transform everything it encounters—numbers, words, special characters, and syntax—into a standard JavaScript function. This simple but powerful logic is the foundation upon which the entire language is built.

The process follows a clear, sequential logic:

1. Read a Token: The process starts with read_word, a utility that reads characters from the source code until it encounters a space or a special character (like (, ), [, ], {, }, \``, or "`). The resulting string is the token.
2. Lookup in Environment: compile_element first searches for the token in the current chain of environments. If the token is a known word, it's already a JavaScript function, which is promptly returned.
3. Fallback to compile_atom: Only if that fails does it fall back to compile_atom. This function uses a series of regular expressions and pattern matching to handle literals and other special syntax. For example, it recognizes the string "1" as a number and converts it into a function that, when executed, pushes the number 1 onto the stack (e.g., () => put(1)).
4. Handle Immediate Words: Some words are marked as "immediate." Instead of being returned as a function for the runtime array, these words are executed directly during compile time. They are used to implement syntax, control flow, and definitions.
5. Trigger an Error: If a token is not a known word and compile_atom cannot recognize it as a literal or special syntax, an error is triggered.

Immediate words, which execute during compilation, are the primary mechanism for extending the language's syntax and behavior. They come in two distinct types:

Type	Behavior
Immediate 0	Executes immediately at compile time. Returns undefined, so nothing is added to the final execution array. Used for side effects like defining a new word.
Immediate 1	Executes immediately at compile time. It is expected to push a function onto the stack. compile_element then pops this function and returns it to be added to the final execution array.

This elegant mechanism—turning tokens into functions or executing them immediately to manipulate the compilation process—is how all syntax and control flow are constructed. The most fundamental use of this system is defining your own words.

4.0 Your First Words: Definitions and Environments

Creating custom words is the primary way a developer extends the "oh" language and builds powerful abstractions. At the center of this process is the concept of an environment, which acts as a dictionary for words.

The Anatomy of an Environment in "oh" is a simple JavaScript object with two keys:

* parent: A pointer to the parent environment, forming a chain for word lookups.
* word: An object that serves as a lookup table, mapping word names (strings) to their definitions (JavaScript functions).

The Colon (:) Word is the primary tool for creating new words. It is an immediate 0 word whose job is to read a sequence of tokens from the source code and compile them into a new word definition. Its syntax is straightforward:

: push-three-numbers 1 2 3 ;


When the compiler encounters :, it executes immediately and performs the following steps during compile time:

1. It reads the next token (push-three-numbers) to use as the name for the new word.
2. It creates a new, temporary environment, setting its parent to the current environment. This new environment becomes the active one for compiling the body of the word.
3. It enters a loop, reading subsequent words (1, 2, 3) until it encounters a semicolon (;), which marks the end of the definition.
4. For each word in the body, it calls compile_element and stores the resulting function in a temporary code array.
5. When the semicolon is reached, it restores the environment pointer back to its parent (the environment that was active before the definition started).
6. Finally, it creates the new word (push-three-numbers) in the parent environment. The definition is a JavaScript function that, when executed at runtime, simply iterates through and calls each function stored in its code array.

Compile-Time Lexical Scope is a key concept here. The temporary environments created by : exist only during compilation. Once a definition is complete, the environment used to compile it is no longer referenced and is eventually garbage-collected. For instance, if a definition is nested inside another, it is only visible to the outer definition during its compilation.

: outer-definition
    : inner-definition 1 2 3 ;  -- inner-definition is only visible here
    inner-definition             -- This freezes inner-definition into outer-definition
;


In this example, inner-definition is defined within the temporary compile-time environment of outer-definition. It can be used within outer-definition's body, but once compilation of outer-definition is complete, that temporary environment disappears, and inner-definition is no longer accessible globally.

While definitions provide structure, managing the data they operate on requires specific tools for interacting with the stack. This is where bindings come into play.

5.0 Managing Data: The bind and declare Words

Stack-based languages can often lead to complex "stack juggling"—the manual reordering of data on the stack to get it in the right place for an operation. To mitigate this and improve code readability, "oh" provides the bind word, which allows you to associate names with values.

The bind word is an immediate 1 word, and its design cleverly bridges the gap between compile time and runtime. It must be immediate because it needs to create a new word (the binding) in the environment at compile time, making that name available for subsequent code. However, the value it will be bound to only exists on the stack at runtime.

Internally, a binding is a JavaScript closure that acts as a "cell," storing its value as a property on the function object itself. The function looks something like this:

const binding = () => put(binding.value);


When this function is executed, it simply pushes its own stored value onto the stack. To see how bind uses this to create readable abstractions, let's deconstruct a definition that uses it:

: complex-operation bind number
    number 1 +
;


This word expects a value on the stack, binds it to the name number, and then adds 1 to it. Let's walk through its compilation and subsequent execution with 24 complex-operation.

Compile Time

* The : word begins compiling complex-operation, creating a new, temporary environment for it.
* The first token it encounters is bind, an immediate 1 word. bind executes immediately during the compilation of complex-operation.
* bind reads the next token, number, and creates the binding function in the current temporary environment.
* Crucially, bind then pushes a "setter" function onto the stack: () => binding.value = stack.pop().
* Because bind is immediate 1, compile_element takes this setter function from the stack and returns it. The : word adds this setter to the code array of complex-operation.
* The : word continues compiling. It reads number, finds the newly created binding in the temporary environment, and adds it to the code array.
* Finally, it compiles 1 and +, adding their functions to the code array. The definition is now complete.

Runtime (24 complex-operation)

* First, the code for 24 runs, pushing the number 24 onto the stack.
* Next, complex-operation runs, executing its compiled code array function by function:
  1. The setter function runs. It pops 24 from the stack and assigns it to the number binding's internal value.
  2. The number binding function runs. It pushes its stored value (24) back onto the stack.
  3. The function for 1 runs, pushing 1. The stack is now [24, 1].
  4. The function for + runs, popping 24 and 1, adding them, and pushing the result 25 onto the stack.

The bind word can also be used with a list of names to bind multiple values from the stack at once, further reducing stack manipulation:

1 2 3 bind (one two three)


For situations where you want to create a binding without an initial value from the stack, "oh" provides declare. It is an immediate 0 word that simply creates a binding in the current environment at compile time and initializes its value to 0.

With tools to define words and manage data, the next step is to build logic and control the flow of execution.

6.0 Extending the Language: Syntax and Control Flow

One of the most distinctive features of "oh" is its lack of built-in syntax. Constructs that are keywords in other languages—like if statements or list literals—are not part of the core interpreter. Instead, nearly all control structures and syntactic sugar are implemented as immediate words that manipulate the compilation process itself.

Parentheses: The Simplest Syntax. The parenthesis ( is a perfect example of syntax extension. It is an immediate 1 word whose job is to create a list. When the compiler sees (, it executes the word immediately. This word then reads all subsequent tokens until it finds a closing ). It collects these tokens into a JavaScript array and pushes a new function onto the stack. This function, when executed at runtime, will push a fresh copy of that array onto the stack. The use of list.slice() ensures that every execution provides a new, mutable copy, preventing side effects.

The if Word: Conditional Logic. The if word is another immediate 1 word that provides conditional logic. It works by compiling three distinct blocks of code: a test condition (between if and then), a true branch (between then and else/end), and an optional false branch (between else and end). At compile time, if reads and compiles these blocks, then returns a single function to the main execution array. At runtime, this single function executes the test condition code, pops the result from the stack, and, based on whether the result is truthy, conditionally executes either the true branch or the false branch.

Delayed Lookup for Forward References. By default, "oh" requires words to be defined before they are used, triggering a compile-time error otherwise. However, the language provides syntax sugar for Delayed Lookup to defer this check from compile time to runtime. By appending a colon to a word (e.g., some-word:), you instruct compile_atom to generate a function that will look up and execute some-word at runtime.

This mechanism serves two primary use cases:

* Forward References: It allows you to call a word before it has been defined in the source code.
* Recursion: It provides a basic, albeit inefficient, way to implement recursion by allowing a word to look up its own name at runtime within its definition.

This mechanism works for top-level definitions due to a crucial detail: at runtime, after all compilation is complete, the current environment pointer almost always points back to the root environment. Therefore, a delayed lookup for a top-level word will successfully find it in root when the code finally executes. This seemingly simple feature, however, exposes a fundamental tension in the language's design that becomes critical when we explore more advanced features.

These extension mechanisms are not just theoretical toys; they are essential for giving "oh" practical power, particularly for interacting with its host environment, such as the browser's Document Object Model.

7.0 Advanced Interoperability: The DOM, Async, and Modules

The practical power of "oh" shines through its deep integration with the JavaScript environment, but this integration introduces a central conflict between the language's core philosophy (compile-time purity) and the practical demands of its goals (DOM manipulation and asynchronous code). This section covers the advanced features that navigate this tension to build user interfaces, handle asynchronous operations, and organize code.

7.1 The dom Word and Runtime Environments

The dom word is a normal runtime word used to build HTML interfaces. It takes a list from the stack—representing a nested structure of elements—and converts it into live DOM nodes.

The list format uses special directives to define attributes, events, and bindings:

* #id-name: Sets the element's id.
* .class-name: Adds a CSS class to the element.
* -attribute value: Sets an arbitrary element attribute (e.g., -type "text").
* @event (code): Attaches an event listener. The (code) list is compiled at runtime, and within it, a special event word is available that pushes the browser's event object onto the stack.
* :word-name: Creates a new word at runtime in the current environment. When executed, this word pushes the newly created DOM element onto the stack.
* writer word-name property and reader word-name property: Create setter and getter words at runtime for a specific property of the DOM element.

The author is candid that this powerful feature, especially the ability to compile code and create words at runtime, creates a central conflict. The dom word necessitates the existence of runtime environments, which contradicts the interpreter's original design goal of avoiding runtime lookups. The current solutions—words like block and defun, combined with delayed lookups (word-name:)—are, in the creator's own words, "hacks" or a "hotfix" to manage this mismatch. They are experimental workarounds that highlight the ongoing evolution of the language's design.

7.2 Asynchronous Operations with wait

To handle JavaScript's asynchronous nature, "oh" provides the wait word. This is an immediate 0 word that "hijacks" the compilation process to enable automatic handling of Promises.

When wait is executed at compile time, it replaces the standard function generator (make_sub) with an asynchronous version (make_async_sub). This has a profound effect: any definitions compiled after wait is called become async JavaScript functions. At runtime, these functions automatically check if a Promise appears on top of the stack after executing a step. If one does, the interpreter awaits its resolution before continuing. This allows for seamless handling of asynchronous APIs like fetch:

wait
'https://api.example.com/data' fetch -json log


A crucial 'gotcha' arises from this mechanism. Placing wait and no-wait within the same definition block will not work as one might intuitively expect. The function that generates the final executable iterator (make_sub or make_async_sub) is only called at the end of the compilation for that block. If no-wait is present, it will revert the setting just before the iterator is created, meaning the entire block will be compiled without the auto-await behavior. This provides a brilliant, practical insight into the compiler's mechanics: the 'mode' must be set for the entire duration of a definition's compilation.

7.3 Code Organization with module and import

To help organize code, "oh" provides a simple module system.

* The module word is an immediate 0 word. It compiles a block of code into a dedicated, named environment. The name of the module itself becomes an immediate word that, when used at compile time, pushes its environment onto the stack.
* import and import-all are immediate words that operate at compile time. They take an environment from the stack (provided by a module word) and copy either specified words or all words from that module's environment into the current compile-time environment.

These advanced features demonstrate how "oh" extends its simple core to tackle complex, real-world programming challenges. The final piece of the puzzle is understanding how to observe and debug this unique execution model.

8.0 Debugging and Introspection: The trace Word

Understanding the state of the stack is crucial in a stack-based language. To aid in this, "oh" includes a simple but powerful debugging tool: the trace word.

The trace mechanism is another clever example of manipulating the compilation process. trace is an immediate 0 word that hijacks the compiler by replacing the standard compile_in_list function with a new version, trace_into_list.

The effect of this replacement is that every subsequent function compiled is wrapped in another function. This wrapper first executes the original code and then logs the original source token and the complete state of the stack after the operation to the console.

For example, running the following code:

trace
1 2 +


Would produce this output, showing the stack's state after each step:

trace: 1 1
trace: 2 1 2
trace: + 3


To stop this behavior, the no-trace word restores the original compile_in_list function. This on-demand approach has a key benefit: the performance overhead of tracing is only incurred when it is explicitly enabled. During normal execution, the interpreter remains lean and fast.

This tutorial has journeyed from the core philosophy of "oh" to its practical applications and debugging tools. It is a language built not for production, but for learning. Every feature, from immediate words to the "hacks" for runtime environments, serves as a tangible experiment in language design. As a "mind playground," "oh" invites you to not only use its concepts but to understand them, question them, and perhaps, be inspired to build your own.
