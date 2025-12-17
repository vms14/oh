# Language Words Reference

## Core Syntax & Structure

### `(`  
Builds a literal list. Reads until `)` and pushes a function that pushes a JS array copy of that list.

### `)`  
Terminator for `(`. Not a word by itself.

### `{`  
Builds a quoted block. Reads until `}` and pushes a function that pushes compiled code.

### `}`  
Terminator for `{`.

### `:` *(immediate)*  
Defines a word. Reads a name, then compiles code until `;`, storing it in the environment.

### `;`  
Implicit terminator for `:` blocks.

---

## Compilation & Evaluation

### `compile`  
Compiles a single element and pushes the resulting function.

### `compile-string`  
Compiles a string into a function and pushes it.

### `eval`  
Evaluates a compiled element taken from the stack.

### `eval-string`  
Interprets and executes a string immediately.

### `block` *(immediate)*  
Compiles a block into a function that executes in a new environment.

### `lambda` *(immediate)*  
Compiles a block and pushes a function that returns a closure when executed.

---

## Control Flow

### `if` *(immediate)*  
Conditional construct. Syntax: `if … then … else … end`.

### `else`  
Error if used outside `if`.

### `end`  
Block terminator. Errors if misplaced.

### `case` *(immediate)*  
Multi-branch dispatch. Builds a value → code map and executes matching code.

### `nop`  
No operation.

---

## Stack Manipulation

### `drop`  
Removes top stack element.

### `2drop`, `3drop`, `4drop`  
Remove N elements from stack.

### `dup`  
Duplicates top element.

### `swap`  
Swaps top two elements.

### `list`  
Pops a number `n`, then pops `n` elements and pushes them as a list.

### `flatten`  
Spreads a list onto the stack.

---

## Arithmetic & Math

### `+`, `-`, `*`, `/`, `%`  
Binary arithmetic operators.

### `..`  
Alias for `+` (string/number concatenation).

### `random`  
Random selection:
- list → random element
- object → random value
- number → random integer < n

---

## Variables & Binding

### `declare` *(immediate)*  
Declares variables with default value `0`.

### `bind` *(immediate)*  
Binds values to names (single or multiple).

### `defun` *(immediate)*  
Defines a named function in the current environment.

---

## Mutation & Accessors

### `set` *(immediate mutator)*  
Sets a variable or property.

### `get` *(immediate mutator)*  
Gets a variable or property.

### `increment`, `decrement` *(immediate mutators)*  
Add/subtract value.

### `increment-by-one`, `decrement-by-one` *(immediate mutators)*  
Unary increment/decrement.

---

## Property / Dot Notation

### `.property`  
Reads property from object on stack.

### `.property!`  
Writes property to object on stack.

### `object.property`  
Evaluates object expression, then accesses property.

### `object.property!`  
Same, but assignment.

### `get-property`  
Explicit property access.

---

## Methods & Calls

### `-method`  
Calls method on object: `obj.method()`.

### `--method`  
Calls method with arguments: `obj.method(...args)`.

### `~method`  
Calls method without pushing return value.

### `~~method`  
Calls method with arguments, discarding result.

---

## Strings & Literals

### `"` *(immediate)*  
Reads a quoted string and pushes a function that returns it.

### `` ` `` *(immediate)*  
Alternative string literal (same mechanics).

### `'string`  
Literal string atom.

---

## Comments

### `---` *(immediate)*  
Line comment. Skips input until newline.

---

## Environment & Modules

### `find`  
Finds a word and pushes it.

### `module` *(immediate)*  
Creates a module (environment) and binds it.

### `import`  
Imports specific words from a module.

### `import-all`  
Imports all words from a module.

---

## Async & Timing

### `promise` *(immediate)*  
Creates a Promise with `resolve` / `reject` available inside.

### `wait` *(immediate)*  
Switches compiler to async mode.

### `no-wait` *(immediate)*  
Restores synchronous compilation.

### `interval` *(immediate)*  
Creates a `setInterval`.

### `timeout` *(immediate)*  
Creates a `setTimeout`.

---

## Iteration

### `iterate`  
Iterates over list, executing code for each element.

### `iterator`  
Creates an iterator object.

### `next-element`  
Moves iterator forward.

### `previous-element`  
Moves iterator backward.

### `random-element`  
Jumps iterator to random index.

---

## Objects & Data

### `object`  
Builds object from key-value list.

### `obj`  
Builds object from keys list + values.

### `properties`  
Pushes values of object properties.

### `keys`, `values`, `entries`  
Object introspection.

### `random-key`  
Random object key.

---

## Text & Utilities

### `uppercase`, `lowercase`  
String casing.

### `remove-characters`  
Removes characters from string.

---

## CSS & DOM (Browser)

### `css`  
Converts CSS AST into CSS string.

### `style`, `style-append`  
Inject CSS.

### `make-canvas`, `canvas`, `set-ctx`  
Canvas setup.

### `rectangle`, `color`, `clear`  
Canvas drawing.

### `dom` *(immediate)*  
DOM builder DSL.

### `handler` *(immediate)*  
Event handler builder.

### `append`, `to-body`  
DOM insertion.

### `image`, `images`  
Async image loading.

### `animation`  
Animation loop.

---

## Debugging & Tracing

### `log`  
Logs top of stack.

### `s`  
Logs entire stack.

### `trace` *(immediate)*  
Enables tracing (wraps compiled code).

### `no-trace` *(immediate)*  
Disables tracing.

---

## Networking & Host Access

### `fetch`  
Pushes fetch Promise.

### `console`  
Pushes JS console object.

---

## Interpolation

### `@`  
Interpolates lists using stack values.

---

## Special Immediate Flags

: wait no-wait defun trace no-trace module import import-all declare --- block lambda bind increment { ( " ` if promise interval timeout case