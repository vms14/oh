Understanding Definitions and Environments in 'oh'

Introduction: The Building Blocks of Abstraction

Welcome to 'oh', a language designed not just for programming, but as a playground for the mind. Its creator describes it as a reflection of their own evolving understanding of programming. By learning its core mechanics, you're not just learning syntax; you're gaining insight into a unique philosophy of language design.

Creating your own words, or "definitions," is the primary way you'll extend the language and build powerful abstractions. To truly master this, there are two core concepts you must grasp:

1. The distinction between compile time and runtime.
2. The structure of environments, which is how 'oh' stores its memory.

Once you understand these two ideas, the way 'oh' handles everything from simple words to complex, nested definitions will become exceptionally clear. Let's begin.


--------------------------------------------------------------------------------


1. The Two Phases of 'oh': Compile Time vs. Runtime

The 'oh' interpreter processes your code in two distinct phases. Understanding this separation is the first and most critical step to understanding how definitions are created and used.

Compile Time

Compile time is the phase where 'oh' first reads your source code. It breaks the code into individual tokens (like 1, 2, +, or some-word-name) and its main goal is to turn each token into a JavaScript function. These functions aren't executed yet; they are collected and stored in an array for later.

Imagine the interpreter sees this line of code: 1 2 3

During compile time, a function called compile_element processes each token. It converts the string "1" into a closure that captures the numeric value. When this closure is executed at runtime, it pushes its captured value onto the stack. It does the same for 2 and 3. The final result of this phase is an "array of functions to be executed later," which you can think of conceptually like this:

// An array of closures, each capturing a value to be used later.
[() => put(1), () => put(2), () => put(3)]


Runtime

Runtime is the phase that happens after all the source code has been processed and compiled into that array of functions. Runtime is simply the process of iterating through the compiled array and calling each function, one by one, in sequence.

This separation is a core design principle of 'oh'. Its main purpose is to "freeze" definitions at compile time. When a word uses another word, the reference to it is found once and baked into the compiled code. This avoids dynamic binding, meaning that if the other word is redefined later, the original word that used it will not be affected. This precomputation of decisions makes the runtime simpler and more predictable.

This separation between compiling functions and running them is key, as some special words, like the one for creating definitions, execute immediately during compile time. But before we look at that, we need to understand where 'oh' stores its words: in environments.


--------------------------------------------------------------------------------


2. Environments: The Memory of 'oh'

An environment is the data structure 'oh' uses to store and look up words. Think of it as the language's memory or vocabulary. Structurally, it's a simple JavaScript object that acts as a link in a chain, with two properties:

{ parent: pointer_to_another_environment_or_undef, word: {} }


Here's a breakdown of its two key components:

* word: This is a lookup table (a simple object) that maps word names (like + or dup) to their corresponding JavaScript functions. This is where the actual definitions are stored.
* parent: This property creates a chain of environments. When the interpreter looks for a word, it first checks the word object of the current environment. If it doesn't find the word there, it follows the parent pointer to the next environment and searches there. This process continues up the chain until it either finds the word or reaches the root environment, which has no parent.

During compilation, the interpreter uses a global pointer, env, to keep track of the current environment it should be searching and writing to.

Now that we understand that words live inside these chained environments, let's see the step-by-step process of how the colon word (:) creates a new word and adds it to an environment.


--------------------------------------------------------------------------------


3. Creating Definitions with the Colon Word (:)

The primary way to create a new definition in 'oh' is with the colon word (:). Its basic syntax is : word-name ... ;, where ... represents the body of the new word.

The most critical insight is that : is an "immediate word" that executes entirely during compile time. Its job is not to be part of the final program's execution, but to perform the side effect of adding a new word to the current environment.

The following table breaks down what happens when 'oh' compiles a definition like : push-three-numbers 1 2 3 ;.

Step	Action	The "Why" for the Learner
1. Execution	The compile_element function sees : and, because it's an immediate word, executes its function right away instead of saving it.	This is a compile-time action. The colon word's purpose is to build another word, not to be part of the final execution array itself.
2. Naming	The : word's function reads the next token from the source (push-three-numbers) to use as the name for the new definition.	Every new word needs a name to be looked up by later.
3. Scoping	It creates a new, temporary environment whose parent is the current environment. This new environment becomes the current one for compiling the definition's body.	This creates a private, temporary "workspace" or "lexical scope" for the new definition. Any words defined inside are only visible during the compilation of this word.
4. Compiling	The : word reads all subsequent tokens (1, 2, 3) until it finds the semicolon (;). It passes each token to compile_element and stores the resulting functions in a code array.	This is the same compilation process from section 1, but the resulting functions are being collected for the new word, not for the main program.
5. Finalizing	Upon reaching ;, the : word restores the original environment as the current one. It then creates the final word—a function that simply iterates and executes the collected code array (conceptually, () => { for (let fun of code) fun() })—and adds it to the environment under the name from Step 2.	The new word is now officially part of the language's vocabulary. The temporary environment is discarded, ensuring a clean scope.

An Inside Look at the Colon Word's Logic

To make this process even more concrete, here is a simplified version of the JavaScript code that implements the : word. You can see how it maps directly to the steps in the table.

// This function is assigned to env.word[":"] and marked as "immediate"
() => {
  // Step 2: Naming
  const name = read_word();

  // A place to store the compiled body of the new word
  const code = [];

  // Step 3: Scoping (Part 1 - Save and switch environment)
  const old_environment_pointer = env;
  env = make_env(env); // Creates the new, temporary environment

  // Step 4: Compiling
  let word;
  while ((word = read_word()) !== ';') {
    const value = compile_element(word);
    if (value) {
      code.push(value);
    }
  }

  // Step 5: Finalizing
  env = old_environment_pointer; // Restore the original environment
  env.word[name] = () => { for (let fun of code) fun() }; // Add the new word
}


This process of creating and discarding temporary environments during compilation is what enables powerful scoping features, including nested definitions.


--------------------------------------------------------------------------------


4. Understanding Nested Definitions

Because the : word creates its own temporary compile-time environment, you can nest definitions inside one another. Let's first look at an example that illustrates the mechanism but is ultimately "useless" to see how the scoping works.

: outer-definition
  : inner-definition 1 2 3 ;
;


Here is what happens step-by-step during compile time:

1. The outer-definition begins compiling. The outer : executes. It saves a reference to the current environment (the root environment) in a temporary variable, then creates a new environment (outer_env) and sets the global env pointer to it.
2. While compiling the body, it encounters the inner :. This inner word executes right away. It saves a reference to the now-current outer_env, creates its own new environment (inner_env), and sets the global env pointer to this new inner_env. It then defines inner-definition within inner_env.
3. The inner definition finishes at its ;. The inner : word restores the env pointer back to its saved reference, outer_env. The inner_env is now unreferenced and will be garbage collected.
4. The outer-definition finishes at its own ; and restores the env pointer to the root environment. The outer_env is also discarded.

Crucially, because inner-definition was never used within the body of outer-definition, its existence was purely temporary and is now lost. The final outer-definition is created as an empty, "do nothing" word.

Now, let's contrast this with a "useful" nested definition:

: outer-definition
  : inner-definition 1 2 3 ;
  inner-definition
;


The key difference happens after the inner definition is created:

1. The process is the same as before: inner-definition is created and added to the temporary outer_env.
2. The outer-definition continues compiling its body and now encounters the token inner-definition.
3. It calls compile_element on this token. The interpreter searches for the word in the current compile-time environment (outer_env) and finds it.
4. The function for inner-definition is returned and "frozen" as a closure in the code array of the outer definition.

This leads to a key insight: A nested definition is only useful if it is referenced within the outer definition's body. This act of referencing it captures the definition before its temporary compile-time environment is destroyed, embedding it forever within the outer word's compiled code.


--------------------------------------------------------------------------------


5. Conclusion: Your Foundation for 'oh'

You now have the foundational knowledge needed to build complex and well-structured programs in 'oh'. Let's recap the three most important takeaways:

1. Compile Time vs. Runtime: 'oh' first compiles source code into a list of functions (compile time) and then executes that list (runtime). Immediate words like : act during compile time to shape the language.
2. Environments Provide Scope: Words are stored in chained environments. The : word creates a new, temporary environment for compilation, providing what is effectively lexical scope at compile time.
3. Nesting Captures Definitions: Nested definitions are only preserved if they are used within the parent definition. This "freezes" a reference to the inner word in the parent's compiled code before the temporary environment disappears.

With this foundation, you are now equipped to do more than just use 'oh'—you are ready to treat it as your own mind playground. The same mechanisms of immediate words and environments you've just learned are your tools for testing ideas, building new syntax, and extending the language to match your own understanding.
