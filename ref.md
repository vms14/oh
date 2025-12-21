### oh Programming Language Reference

Foreword

This document is a comprehensive reference manual for the oh programming language, designed for professional developers. oh is a stack-based, reverse-polish notation (RPN) language written in and deeply integrated with JavaScript. It is intended as a "mind playground"—a minimalist, extensible environment for experimenting with, testing, and evolving programming concepts. This manual details its core philosophy, mechanics, and features, providing the necessary knowledge to leverage its unique approach to programming.

### 1.0 Core Philosophy and Concepts

Understanding a language's core philosophy is crucial for mastering its use. The design of oh prioritizes simplicity, directness, and profound integration with its host environment (JavaScript). Its central tenet is that every token represents an executable action. This principle governs a two-phase operational model—"compile time" and "runtime"—that resolves decisions early to ensure a predictable and efficient execution phase.

1.1 Design Goals

The creation of oh was driven by a distinct set of objectives that define its character and utility.

* JavaScript Interoperability: A primary goal is to inherit the full power of the host environment. oh is designed to have complete access to browser and Node.js APIs, allowing it to do everything JavaScript can do, albeit in its own unique style.
* Conceptual Playground: The language serves as a personal laboratory for the author to experiment with programming ideas. Its simple, extensible nature allows for the rapid testing, integration, and evolution of new concepts with minimal friction.
* A Reflection of Understanding: oh is a living embodiment of the author's programming knowledge. The interpreter's core and its library of "words" (functions) grow and are rewritten as the author's understanding of programming concepts deepens and matures.

1.2 The "Compile Time" vs. "Runtime" Duality

The central operational concept in oh is a strict separation between two phases: compile time and runtime. This is not a traditional compilation to bytecode or machine code; rather, it is a process of organizing and delaying execution.

* Compile Time is the process of reading source code tokens and converting them into an ordered array of JavaScript functions. During this phase, word lookups are performed, syntax is resolved, and definitions are effectively finalized. While this pre-computation minimizes runtime overhead, its primary philosophical purpose is to freeze definitions. By resolving a word's meaning when it is compiled into a definition, oh avoids dynamic binding. This ensures that a function's behavior remains stable and predictable, even if the words it depends on are later redefined.
* Runtime is the subsequent phase where the interpreter iterates through the array of functions generated during compile time and executes each one in sequence. This is the stage where the actual work of the program—manipulating the stack, performing calculations, interacting with the host environment—is carried out.

1.3 The Evaluation Loop

Whether running in a Read-Eval-Print Loop (REPL) or loading a file, the oh interpreter follows a consistent four-step process:

1. Read: The internal read_word() function consumes tokens from the source string. Tokens are typically separated by whitespace, but certain special characters are treated as standalone tokens.
2. Compile: The compile_element() function, the heart of the interpreter, receives each token. It attempts to resolve the token to a known function ("word") in the current environment. If it's an "immediate" word, it's executed instantly; otherwise, the function is prepared for collection.
3. Collect: If compile_element() returns a function, it is appended to an execution array. Immediate words that do not return a function are executed for their side effects (like defining a new word) but are not added to this array.
4. Execute: Once the source code is fully consumed, the interpreter iterates through the collected array, calling each function in order. This constitutes the runtime phase.

This simple yet powerful loop forms the foundation upon which all of the language's features and mechanics are built.

### 2.0 Interpreter Mechanics

This section dissects the engine of the oh language. It covers the fundamental components—the stack, environments, and compilation functions—that govern how code is resolved and executed, providing a deeper understanding of the "compile time" and "runtime" duality.

2.1 The Stack

The stack is the central data workspace in oh. It is a Last-In, First-Out (LIFO) data structure where all intermediate values are stored. Nearly every word in the language operates by consuming one or more values from the top of the stack, performing an action, and pushing one or more results back onto the stack. Effective oh programming requires developing a mental model of the stack's state at each step of execution.

2.2 Environments and Scoping

An oh environment is a simple JavaScript object that provides the mechanism for scoping. Each environment contains two keys:

* parent: A reference to the parent environment, forming a chain that terminates at the root environment (where parent is undefined).
* word: An object that serves as a lookup table, mapping word names (strings) to their corresponding JavaScript functions.

The interpreter maintains a global env pointer that tracks the current environment for all word lookups.

Compile-Time Lexical Scope

oh employs a form of lexical scoping that is resolved entirely at compile time. When a new definition is created (e.g., using the : word), a new, temporary environment is established with the current environment as its parent. This new environment exists only for the duration of that definition's compilation.

When the definition is complete, the temporary environment is discarded. However, any words that were looked up during its compilation have already been resolved and "frozen" as direct function references within the new word's internal execution array. This prevents dynamic binding; if a word used in a definition is later redefined, the original definition remains unaffected because it holds a direct reference to the old function, not a name to be looked up again.

2.3 The Compilation Pipeline

The conversion of source tokens into executable functions is handled by a two-stage pipeline.

compile_element()

This function is the main entry point for the interpreter's compilation logic. For any given token, it performs the following steps:

1. It searches the environment chain, starting from the current env, for a word matching the token.
2. If a word is found:
  * If the word is marked as "immediate," it is executed immediately at compile time. Depending on its type, it may return a function to be collected or nothing at all.
  * If it is a standard word, the function itself is returned to be collected.
3. If no word is found, the token is passed to compile_atom() for further processing.

compile_atom()

This function acts as the "syntax sugar" layer, handling tokens that are not defined words. It uses a series of checks, primarily regular expressions, to recognize and process common language constructs:

* Numbers: Numeric tokens are converted into functions that, at runtime, push the corresponding number onto the stack.
* Simple String Literals: As a syntactic shortcut, tokens prefixed with a single quote (e.g., 'hello) are compiled into functions that push the subsequent string onto the stack.
* Delayed Lookups and Bindings: Tokens containing colons (:) are compiled into functions that perform word lookups or create new words at runtime instead of compile time.
* JavaScript Property Access: Tokens containing dots (.) are compiled into functions for getting or setting properties on JavaScript objects.
* JavaScript Method Calls: Tokens prefixed with -, --, ~, or ~~ are compiled into functions that call methods on JavaScript objects.

If compile_atom cannot recognize a token, it triggers a "not recognized" error, halting compilation. This strictness ensures that all tokens have a well-defined action.

From these internal mechanics, we can now explore the language features that developers interact with directly.

### 3.0 Language Fundamentals

This section covers the fundamental syntax and constructs used to write oh programs. It focuses on how developers define words, create literal values, and leverage the immediate word system to extend the language's syntax.

3.1 Tokenization

The read_word function is responsible for parsing source text into a stream of tokens. Its rules are straightforward:

* Tokens are primarily delimited by one or more whitespace characters (spaces, newlines, or tabs).
* A set of special characters are treated as standalone tokens, even without surrounding whitespace. These are: (, ), [, ], {, }, \``, and "`.

For example, the string (1 2 3) is tokenized identically to ( 1 2 3 ), producing five distinct tokens: (, 1, 2, 3, and ).

3.2 Word Definitions with : and ;

The primary mechanism for creating a new word is the : (colon) word, which is always terminated by a ; (semicolon).

1. The : word is an immediate 0 word, meaning it executes at compile time.
2. It immediately reads the next token from the source stream, which becomes the name of the new word being defined.
3. It then enters a compilation loop, reading all subsequent tokens and compiling them into an internal execution array until it encounters the ; delimiter.
4. Once ; is found, it creates a final function that encapsulates the execution of this compiled array.
5. This new function is then added to the word lookup table of the current environment under the specified name.

For example, the definition : push-three-numbers 1 2 3 ; creates a new word named push-three-numbers. When push-three-numbers is later executed, it will run the compiled functions for 1, 2, and 3 in sequence, resulting in those three numbers being pushed onto the stack.

3.3 Immediate Words

Immediate words are the key to oh's extensibility. Unlike standard words, which are collected into an execution array to be run later, immediate words are executed by the compiler the moment they are encountered. This allows them to alter the compilation process itself, effectively creating new syntax. There are two types of immediate words.

Type	Flag	Behavior	Primary Use Case
Immediate 0	0	Executes at compile time. compile_element returns undefined.	Performing compile-time side effects, such as defining a word (:) or changing a compiler setting (trace).
Immediate 1	1	Executes at compile time and must push a function onto the stack. compile_element pops this function and returns it.	Creating new syntax that produces a runtime action. This is used for literals ((), control flow (if), and bindings (bind).

3.4 Literals

Literals are fundamental values like numbers, strings, and lists. In oh, they are created by immediate 1 words that generate and return functions.

* Numbers: Any token that matches the number regex (e.g., 42, -3.14) is handled by compile_atom. It returns a function that, when executed at runtime, will push the parsed numeric value onto the stack.
* Strings: The " and \`` words are powerful immediate 1string readers. When encountered, they consume all characters from the source stream until a matching closing delimiter is found. They can process escape sequences (e.g.,\n, \t) and return a function that, when executed at runtime, pushes the resulting string onto the stack. This mechanism is more robust than the simple 'prefix handled bycompile_atom`.
* Lists: The ( word is an immediate 1 word that reads and compiles all subsequent tokens until a matching ) is found. It returns a function that, when executed, pushes a fresh copy of the compiled list of values onto the stack. This copy-on-execution behavior is crucial, as it prevents mutations to a list in one part of the code from affecting another part that uses the same literal definition.

With these fundamentals, we can now examine how oh manages state through its binding system.

### 4.0 Bindings and State Management

oh does not have traditional variables. Instead, it offers "bindings"—specialized words that act as mutable cells. This mechanism allows developers to associate names with values, reducing the need for complex stack manipulation ("juggling") in larger definitions.

4.1 bind and declare

bind

The bind word is the primary tool for creating a stateful binding. It is an immediate 1 word that orchestrates a compile-time action and a runtime action.

1. Compile-Time Action: bind reads the next token (or a parenthesized list of tokens) as the name(s) for the new binding(s). For each name, it creates a "binding function"—a closure that, when executed, pushes its internal binding.value onto the stack. This new word is immediately registered in the current compile-time environment, making it available for subsequent code in the same definition.
2. Runtime Action: bind then returns a separate "setter" function to the compilation array. At runtime, this setter function executes, popping a value (or multiple values) from the stack and storing it in the binding.value property of the corresponding closure.

Example:

42 bind the-answer  ( Compiles the number 42, then runs 'bind' )
the-answer          ( Pushes 42 onto the stack )


During compilation, bind creates the word the-answer. It then adds a setter function to the execution array. At runtime, 42 is pushed to the stack, and then the setter function runs, popping 42 and storing it inside the the-answer closure.

declare

The declare word is a simpler variant. It is an immediate 0 word that creates one or more bindings in the current compile-time environment, just like bind. However, instead of generating a runtime setter, it initializes the internal value of each binding directly to 0. This is useful for creating a binding that will be mutated later with words like set.

4.2 set and Other Mutators

The set word is the primary mechanism for changing the value of an existing binding or a JavaScript object property. It is an immediate 1 word.

At compile time, set reads the next token, which must be the name of a binding or a JS property accessor. It finds this target and returns a function to the compilation array. At runtime, this returned function executes, popping a single value from the stack and updating the target binding's internal value or the specified object property.

Several other specialized mutator words are available:

* get: Pushes the current value of a binding or property onto the stack.
* increment: Pops a value and adds it to the target.
* decrement: Pops a value and subtracts it from the target.
* increment-by-one: Increments the target's value by one.
* decrement-by-one: Decrements the target's value by one.

4.3 Understanding Scope

Bindings in oh exhibit a unique blend of lexical and dynamic behavior.

* The lookup of a binding is statically and lexically scoped. When compiling code, the interpreter finds a binding by searching up the compile-time environment chain. If an inner definition creates a binding with the same name as one in an outer scope, the inner binding will "shadow" the outer one for all code within that inner definition.
* The value of a binding is dynamic. Because a binding is a mutable cell (a function object with a value property), any part of the code that holds a reference to that binding function can modify its value via a mutator. This change is immediately visible to all other parts of the code that reference the same binding.

For example:

1 bind counter

: increment-and-report
  increment-by-one counter  ( Mutates the outer 'counter' )
  counter                   ( Pushes the new value )
;

: shadow-and-report
  100 bind counter          ( Creates a new, local 'counter' )
  counter                   ( Pushes 100 )
;

increment-and-report  ( Stack will have [2] )
counter               ( Stack will have [2, 2] )
shadow-and-report     ( Stack will have [2, 2, 100] )
counter               ( Stack will have [2, 2, 100, 2]. The outer counter is unaffected )


This powerful system has implications for advanced patterns like recursion, motivating the introduction of more sophisticated language features.

### 5.0 Advanced Language Features

This section explores features that extend the core capabilities of oh, enabling runtime scoping, modularity, and asynchronous operations. These features often challenge the language's original design assumption of compile-time-only environments and represent areas of active evolution.

5.1 Delayed Lookups and Runtime Word Creation

To enable forward references and a basic form of recursion, oh provides special syntax processed by compile_atom that defers actions from compile time to runtime.

Syntax	Type	Compile-Time Action	Runtime Action
word:	Delayed Lookup	Returns a function that will look up word in the runtime environment.	Finds word in the current env and executes it. Throws an error if not found.
:word	Delayed Binding	Returns a function that will create a binding for word in the runtime environment.	Pops a value from the stack and creates a word named word that pushes this value.
:word:	Delayed Binding+Push	Returns a function that combines the behavior of :word and word:.	Creates the word binding with a value from the stack, then pushes that same value back.

This mechanism is the key to breaking the strict compile-time resolution order, allowing code to refer to words that have not yet been defined.

5.2 Runtime Environments with block and defun

The block and defun words were introduced to provide a form of runtime lexical scope. It is important to understand that these features represent a pragmatic evolution of the language, introduced primarily to support the complex state management required by the dom word. They challenge the core design philosophy of compile-time-only environments, creating a paradigm the interpreter was not originally designed to handle. This reliance on runtime environments represents an area of active development, pushing the boundaries of the language's original assumptions.

* block ... end: block is an immediate 1 word. At compile time, it compiles all the code between block and its corresponding end delimiter. It then returns a single function to the execution array. At runtime, this function creates a new environment, executes the compiled code within that new environment, and finally restores the previous environment.
* defun name ... end: This word is similar to block but creates a reusable, named word. At compile time, it reads a name and compiles the code until end. The resulting word, when called at runtime, performs the environment creation and code execution, effectively creating a function with its own private scope for each invocation.

Words created or accessed within these blocks must use delayed lookups (e.g., word:) because the environments they exist in are only created at runtime, long after the initial compilation has finished.

5.3 Asynchronous Programming with wait

oh includes a built-in "auto await" feature for simplifying asynchronous programming. This is controlled by the wait word.

* wait is an immediate 0 word, so it executes at compile time. Its sole purpose is to swap the standard code execution function (make_sub) with an async version (make_async_sub).
* This async executor runs each compiled function and then checks if the value at the top of the stack is a JavaScript Promise.
* If it is, the executor awaits the Promise, pops it, and pushes the resolved result back onto the stack before continuing to the next function.

This turns synchronous-looking oh code into an asynchronous sequence. The no-wait word reverts the behavior, restoring the standard, non-awaiting executor.

A Note on Compilation Timing: The choice between the standard or async executor is finalized only once, at the end of a compilation block (e.g., when a ; is reached, or at the end of a file). Consequently, a wait and no-wait pair within the same definition block will not behave as a toggle. The state set by the final directive encountered before the block is sealed will apply to the entire compiled unit. This is a crucial detail to remember when managing asynchronous contexts.

5.4 Modules

oh provides a simple but effective module system for organizing and reusing code, which operates entirely at compile time.

* module name ... end: The module word is an immediate 0 word. It reads a name and compiles the block of code between module and end into a new, dedicated environment. It then creates a new immediate 0 word with the given name. When this module name is used, its compile-time action is to push its entire environment object onto the stack.
* import and import-all: These are immediate 0 words that operate on an environment from the stack (typically provided by a module word).
  * import-all pops the environment and copies every word from it into the current compile-time environment.
  * import pops the environment and then reads the next token(s) from the source, copying only the specified word(s) into the current environment.

This system allows for controlled namespacing and code reuse without polluting the global scope.

These advanced features build upon the core mechanics to bridge into the practical domain of JavaScript interoperability.

### 6.0 JavaScript Interoperability

One of oh's primary design goals is seamless interoperability with its JavaScript host. This section details the syntax for interacting directly with JavaScript objects, properties, and methods from within oh code.

6.1 Dot Notation for Properties

oh provides a concise dot notation, processed by compile_atom, for getting and setting properties on JavaScript objects.

Syntax	Operation	Description
.property	Getter (from stack)	Pops an object from the stack, accesses property, and pushes the result.
word.property	Getter (from word)	Executes word, takes the resulting object from the stack, accesses property, and pushes the result.
.property!	Setter (from stack)	Pops an object, then a value. Sets object.property = value.
word.property!	Setter (from word)	Executes word, pops a value. Sets word_result.property = value.

Note: Property names written in kebab-case (e.g., text-content) are automatically translated to camelCase (textContent) during compilation.

6.2 Method Call Syntax

Calling JavaScript methods is handled with a simple prefix-based syntax, also processed by compile_atom. Different prefixes control whether the method's return value is kept and whether arguments are passed from the stack.

Syntax	Pushes Return Value?	Consumes Arguments?	Description
-method	Yes	No	Pops an object, calls obj.method(), and pushes the return value.
~method	No	No	Pops an object and calls obj.method(). Discards the return value.
--method	Yes	Yes	Pops an object and an arguments array. Calls obj.method(...args) and pushes the result.
~~method	No	Yes	Pops an object and an arguments array. Calls obj.method(...args). Discards the result.

These powerful interoperability features are most prominently used for interacting with the browser's Document Object Model.

### 7.0 DOM Manipulation

oh provides a powerful, declarative abstraction for creating and manipulating the browser's Document Object Model (DOM). This system is built upon the core JavaScript interoperability features, offering a concise, s-expression-based syntax.

7.1 The dom Word

The dom word is a standard runtime word that serves as the entry point to the DOM generation system. It expects a list (an s-expression) on the stack, which it interprets as a specification for a DOM tree. It processes this list and pushes the resulting root DOM element back onto the stack.

A typical usage pattern is (p "oh...") dom to-body, which creates a <p> element with the text "oh...", processes it with dom, and appends the resulting element to the document body.

7.2 dom List Directives

Within a dom list, special tokens and patterns act as directives that control the creation and configuration of the DOM element.

Directive	Example	Description
#id-name	(h1 #title "...")	Sets the id attribute of the element.
.class-name	(p .important "...")	Adds a class to the element's classList.
-attribute	(input -type text)	Sets an attribute on the element. The next item in the list is used as the value.
(nested-element...)	(div (p "text"))	A nested list is processed recursively by dom and the resulting element is appended as a child.
"string"	(p "Hello")	A string literal is appended to the element's textContent.
@event-name	(button @click (..))	Adds an event listener. The next item must be a compiled function or a list of 'oh' code to be compiled into the event handler. A local event word is made available.
:word-name	(button :my-button)	Creates a word at runtime that, when executed, pushes the corresponding DOM element onto the stack. Requires a delayed lookup (my-button:) to use.
reader name prop	(p reader p-text text-content)	Creates a "getter" word (p-text) that pushes the value of a property (textContent) onto the stack when called.
writer name prop	(p writer to-p text-content)	Creates a "setter" word (to-p) that pops a value from the stack and sets a property (textContent) on the element.

Debugging these complex, often runtime-generated, interactions requires specific tools built into the language.

### 8.0 Debugging and Diagnostics

oh includes simple but effective debugging tools that are integrated directly into the interpreter's compilation process.

8.1 trace and no-trace

The trace word provides a step-by-step view of a program's execution.

* trace is an immediate 0 word that hijacks the compilation process.
* When active, it wraps every subsequently compiled function in a logger. At runtime, after each original function executes, this logger prints the source token being executed and the complete state of the stack to the console.
* This provides an invaluable, real-time view of how the stack is being transformed by the program's logic.

The no-trace word, also an immediate 0 word, restores the original, non-logging compilation behavior, turning the tracer off.

8.2 Error Conditions

Developers using oh will primarily encounter three types of errors:

* Stack Underflow: This runtime error occurs when a word attempts to pop more items from the stack than are currently available. This is the most common bug in stack-based programming.
* Unrecognized Word: This compile-time error occurs if compile_element and compile_atom cannot resolve a token to any known word, literal, or syntactic form.
* Delimiter Mismatch: This compile-time error occurs when a word that reads a block of code (like (, :, or if) reaches the end of the source without finding its required closing delimiter (e.g., ), ;, end).

The following appendix provides a comprehensive reference to the core words available in the oh language.

9.0 Core Word Reference

This section provides a detailed reference for the standard words built into the oh interpreter. Stack effects are described using the notation before -- after, where items are listed with the top of the stack to the right.

9.1 Stack Manipulation

Word	Stack Effect	Description
dup	a -- a a	Duplicates the top item on the stack.
swap	a b -- b a	Swaps the top two items on the stack.
drop	a --	Removes the top item from the stack.
2drop	a b --	Removes the top two items from the stack.
r	... --	Resets the stack, clearing all items.
s	--	Logs the current state of the stack to the console.
list	a b ... n count -- (a b ... n)	Collects count items from the stack into a new list.
flatten	(a b c) -- a b c	Takes a list from the stack and pushes its elements individually.

9.2 Arithmetic

Word	Stack Effect	Description
+	a b -- a+b	Adds the top two numbers.
-	a b -- a-b	Subtracts the top number from the second number.
*	a b -- a*b	Multiplies the top two numbers.
/	a b -- a/b	Divides the second number by the top number.
%	a b -- a%b	Computes the modulo of the second number by the top number.

9.3 Bindings & Mutators

Word	Stack Effect	Description
bind	val --	(immediate 1) Binds a value from the stack at runtime to a name read at compile time.
declare	--	(immediate 0) Creates a binding with a name read at compile time, initialized to 0.
set	val --	(immediate 1) Sets a binding or JS property to a value from the stack.
get	-- val	(immediate 1) Pushes the value of a binding or JS property onto the stack.
increment	val --	(immediate 1) Adds a value from the stack to a binding or JS property.
decrement	val --	(immediate 1) Subtracts a value from the stack from a binding or JS property.
increment-by-one	--	(immediate 1) Increments a binding or JS property by 1.
decrement-by-one	--	(immediate 1) Decrements a binding or JS property by 1.

9.4 Control Flow & Logic

Word	Stack Effect	Description
if	flag --	(immediate 1) Compiles a conditional block. At runtime, consumes a flag and executes the then branch if truthy, otherwise executes the else branch.
case	val --	(immediate 1) Compiles a multi-branch conditional. At runtime, consumes a value and executes the code associated with that matching case.
eval	code -- ?	Executes a compiled oh function from the stack.
eval-string	str -- ?	Compiles and executes an oh source string from the stack.
iterate	list code --	Iterates over a list, pushing each element onto the stack and then executing the provided code function for each element.

9.5 Compilation & Environments

Word	Stack Effect	Description
:	--	(immediate 0) Begins a new word definition, terminated by ;.
block	-- code	(immediate 1) Compiles a block until end and returns a function that executes the block in a new runtime environment.
defun	--	(immediate 0) Defines a named word that executes its body in a new runtime environment each time it is called.
lambda	-- code	(immediate 1) Compiles a block until end and returns a function that, when called, pushes a new runnable function (with runtime scope) onto the stack.
compile	val -- code	Compiles a value (string, list, etc.) from the stack into an executable oh function.
compile-string	str -- code	Compiles an oh source string from the stack into an executable oh function.
end	n/a	Word used as a delimiter for blocks (block, defun, if, etc.). Throws an error if found elsewhere.

9.6 List, String & Object Operations

Word	Stack Effect	Description
pop	list -- item	Pops and returns the last element from a list.
shift	list -- item	Removes and returns the first element from a list.
object	(k1 v1 k2 v2) -- obj	Creates a JavaScript object from a flat list of key-value pairs.
obj	(k1 k2) v1 v2 -- obj	Creates a JavaScript object from a list of keys and a corresponding number of values from the stack.
keys	obj -- (keys)	Pushes a list of an object's keys onto the stack.
values	obj -- (values)	Pushes a list of an object's values onto the stack.
entries	obj -- (entries)	Pushes a list of an object's [key, value] pairs onto the stack.
..	str1 str2 -- str1+str2	Concatenates two strings. An alias for +.
uppercase	str -- STR	Converts a string to uppercase.
lowercase	str -- str	Converts a string to lowercase.
@	list -- interpolated_list	Takes a list and interpolates values from the stack where , or ,word markers are found.

9.7 Asynchronous & Timing

Word	Stack Effect	Description
wait	--	(immediate 0) Enables "auto await" mode for subsequent compilations.
no-wait	--	(immediate 0) Disables "auto await" mode.
promise	-- code	(immediate 1) Compiles a block that will be executed inside a new Promise constructor, with resolve and reject words made available.
fetch	url -- promise	Wraps the JavaScript fetch function.
interval	-- intervalID	(immediate 1) Reads a time in ms, compiles a block, and creates a setInterval that executes the block.
timeout	-- timeoutID	(immediate 1) Reads a time in ms, compiles a block, and creates a setTimeout that executes the block.

9.8 Modules

Word	Stack Effect	Description
module	--	(immediate 0) Defines a module, compiling its body into a private environment. Creates an immediate word that pushes this environment.
import	env --	(immediate 0) Pops an environment, reads word name(s), and copies them into the current compile-time environment.
import-all	env --	(immediate 0) Pops an environment and copies all its words into the current compile-time environment.

9.9 Browser & DOM

Word	Stack Effect	Description
document	-- document	Pushes the browser document object.
window	-- window	Pushes the browser window object.
body	-- body	Pushes the document.body object.
element	tag_name -- element	Creates a new DOM element of the given tag type.
dom	list -- element	Processes a list as an s-expression and creates a DOM tree.
to-body	element --	Appends a DOM element to the document.body.
append	child parent --	Appends a child element to a parent element.
image	url -- promise	Creates a new Image object and returns a promise that resolves when it loads.
images	(urls) -- promise	Takes a list of URLs and returns a promise that resolves with a list of loaded images.
css	list -- css_string	Converts a list of s-expressions into a CSS string.

9.10 Miscellaneous

Word	Stack Effect	Description
log	a --	Logs the top item of the stack to the console.
nop	--	No operation. Does nothing.
trace	--	(immediate 0) Enables trace mode for subsequent compilations.
no-trace	--	(immediate 0) Disables trace mode.
---	--	(immediate 0) Starts a single-line comment. Ignores all characters until the next newline.
console	-- console	Pushes the browser console object onto the stack.
