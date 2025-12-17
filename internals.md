

# Internal Architecture of the Language

This document explains **how the language actually works**.
Not what it feels like. Not how to sell it.
Only what exists and how it behaves.

The entire system reduces to one idea:

> **Everything is compiled into functions.**

There is no AST.
There is no bytecode.
There is no VM.

Only closures.

---

## Global State

### The Stack

The stack is a single mutable array.

```js
const stack = []
```

There is exactly one stack. It exists at runtime. It is not passed around. It is mutated directly.

All words assume the stack exists. All words implicitly read from and write to it.

This design is intentional:

it removes plumbing

it keeps stack effects explicit

it allows tracing at any point



---

Stack Primitives

Typical primitives look like:

```js
const put = x => stack.push(x)
const get = () => stack.pop()
```

These functions are not language words. They are host-level helpers used by compiled closures.

Language words ultimately compile into closures that call these helpers.


---

Environments

Purpose

An environment maps names → word objects.

A word object minimally contains:

a function

metadata (immediate or not, name, etc.)


Environments answer a single question:

> “Given this token, what function does it represent right now?”




---

Structure

An environment is a linked structure:

```js
{
  word: {}
  parent: Env | null
}
```

Lookup proceeds upward until:

a word is found

or the root environment is reached


This enables:

shadowing

scoped definitions

compiler extension



---

Environment Creation

```js
function makeEnv(parent = null) {
  return { word: {}, parent }
}
```

There is:

a root environment

child environments created as needed


Compile-time and run-time may use different environments, but structurally they are identical.



---

Compilation Model

Tokens

Input is a stream of tokens:

symbols

numbers

strings

control words


The compiler reads one token at a time.

There is no lookahead. There is no grammar. Only dispatch.


---

compileElement (The Core)

compileElement is the central function of the entire system.

Conceptually:

compileElement(token) -> function

Given a token, it returns a closure.

That closure may:

manipulate the stack

call other closures

do nothing

be executed immediately


Every feature in the language funnels through this function.


---

Behavior by Token Type

Literals

If the token is a literal (number, string, etc):

compileElement(42)
→ () => put(42)

No evaluation happens at compile time. The value is captured in the closure.


---

Normal Words

If the token is a word name:

1. Lookup the word in the environment


2. Inspect its immediate flag



If not immediate:

return its function


If immediate:

execute it now

possibly return another function




---

Immediate Words

Immediate words are executed during compilation.

They may:

modify the environment

return a function to be inserted into the output

consume upcoming tokens


Immediate words are how the compiler extends itself.


---

Compilation Output

The compiler produces an array of functions:

[
  fn1,
  fn2,
  fn3,
  ...
]

This array is itself wrapped in a closure:

() => {
  for (const fn of compiled) {
    fn()
  }
}

That wrapper is the final executable program.


---

Definitions

Definitions are implemented as immediate words.

When a definition starts:

1. Compiler switches into “collecting” mode


2. Tokens are compiled into functions


3. Collection ends at a terminator


4. Functions are wrapped into a single closure


5. Closure is stored in the environment under a name



There is no special syntax. Only controlled compilation.


---

Compile-Time vs Run-Time (Precisely)

Compile Time

Tokens are read

Words are resolved

Immediate words execute

Functions are created

Environments may change


Run Time

Only closures execute

No new words are created

Stack effects occur

Environment lookup does not happen


Compile time is not separate execution. It is simply execution in a different phase.


---

Tracing

Tracing is implemented by wrapping functions.

Given a function:

fn

Tracing produces:

() => {
  print(name, stack)
  fn()
  print(name, stack)
}

This works because:

everything is a function

execution is linear

stack is global


No special support is needed.


---

Why This Works

The language works because it obeys these constraints:

1. No hidden state


2. No implicit control flow


3. Everything is explicit closures


4. Compilation is execution


5. Execution is just function calls



There is no magic. Only composition.


---

Invariants

These must always remain true:

Words never directly execute each other

The compiler never executes non-immediate words

Runtime never modifies environments

Stack effects are always visible

No word knows “where it is”


