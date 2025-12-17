
# A Minimal Stack Language That Compiles to Functions

This language turns **words into functions**.

Programs are not executed directly.  
They are **compiled into functions**, and *those functions* are what run.

Some words run **while compiling** and can change how compilation works.
Others only run **when the resulting program is executed**.

That distinction is the core of the language.

---

## The Stack

The language operates on a single global stack.

Conceptually, the stack is just a list of values:

[]

Words consume values from the stack and may push new values onto it.

Examples (stack shown in comments):

1        # [1] 2        # [1, 2]

# [3]


The stack is **not syntax** — it is **state**.
Every word implicitly reads from and writes to it.

---

## Words

A **word** is a named operation.

Internally, every word is represented as a **function**.
When a word is executed, its function runs.

Some words:
- push values onto the stack
- consume values from the stack
- return functions instead of running immediately

You do not call functions directly.
You write words, and the system decides *when* they run.

---

## Environments

An **environment** maps word names to their meanings.

In other words:

> An environment answers the question  
> **“What does this word mean right now?”**

When the compiler sees a word:
1. It looks up the word in the current environment
2. It retrieves a function
3. It decides whether to execute it now or later

Environments are **hierarchical**.
If a word is not found, the lookup continues upward.

This allows new words to override old ones without destroying them.

---

## Compile Time vs Run Time

The language has two phases:

### Compile Time
- Words are **read**
- Functions are **generated**
- Some words execute immediately
- The compiler itself can be extended

### Run Time
- The compiled function is executed
- Stack effects actually happen
- No new words are created

This is not a traditional compiler.
“Compile time” here simply means:
> *the moment words are being turned into functions*

---

## Compile Element (The Core Mechanism)

The heart of the language is a function often called `compileElement`.

Its job is simple:

> Given a word, return a function.

That function may:
- push values
- call other functions
- manipulate the stack
- do nothing at all

Every feature of the language is built on this mechanism.

Numbers, words, definitions, and even control structures
all ultimately compile into functions.

---

## Immediate Words

Some words execute **during compilation**.

These are called **immediate words**.

Instead of returning a function to be executed later,
they *run immediately* and can:
- add new words to the environment
- return new functions to the compiler
- change how subsequent words are compiled

Immediate words are how the language extends itself.

Definitions, macros, and control structures
are all implemented this way.

---

## Definitions

A definition is a word that creates other words.

Conceptually:

1. The compiler encounters a definition word
2. It switches into “collecting” mode
3. It gathers words into a list
4. That list is compiled into a function
5. The function is stored in the environment

From then on, using that word executes the compiled function.

Definitions do not add syntax.
They add **behavior**.

---

## Comments and Stack Traces

Comments are ignored by the compiler.

They are used in documentation to show stack state.

Example:

1 2 +      # [3] 3 *        # [9]

Anything inside a comment is **not code**.
It exists only to help humans understand execution.

---

## Mental Model

If you remember only one thing, remember this:

> The language does not execute words.  
> It **compiles words into functions**,  
> and those functions may run now or later.

