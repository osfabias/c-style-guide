# C Style Guide

Here's the Osfabias C style guide that introduces a set of rules that must be followed when you're writing a C code.

## Rules

### 1. Keep control flow simple 💋

**Rule**: Do not use `goto` statements, `setjmp` or `longjmp` constructs, and direct or indirect recursion.

**Rationale**: Simpler control flow translates into stronger capabilities for verification and often results in improved code clarity. Without recursion we are guaranteed to have an acyclic function call graph, which can be exploited by code analyzers, and can directly help to prove that all executions that should be bounded are in fact bounded. (Note that this rule does not require that all functions have a single point of return - there are cases where an early error return is the simpler solution.)

### 2. Set an upper-bound for loops 🚫

**Rule**: All loops (except loops, that are *meant to be non-terminating*) must have a fixed upper-bound. It must be trivially possible for a checking tool to prove statically that a preset upper-bound on the number of iterations of a loop cannot be exceeded. If the loop-bound cannot be proven statically, the rule is considered violated.

**Rationale**: The absence of recursion and the presence of loop bounds prevents runaway code. One way to support the rule is to add an explicit upper-bound to all loops that have a variable number of iterations (e.g., code that traverses a linked list). When the upper-bound is exceeded an assertion failure is triggered, and the function containing the failing iteration returns an error.

### 3. Allocate memory only on initialization 🏁

**Rule**: Do not use dynamic memory allocation after initialization.

**Rationale**: Memory allocators, such as `malloc`, and garbage collectors often have unpredictable behavior that can significantly impact performance. A notable class of coding errors also stems from mishandling of memory allocation and free routines: forgetting to free memory or continuing to use memory after it was freed, attempting to allocate more memory than physically available, overstepping boundaries on allocated memory, etc. Forcing all applications to live within a fixed, pre-allocated, area of memory can eliminate many of these problems and make it easier to verify memory use. Note that the only way to dynamically claim memory in the absence of memory allocation from the heap is to use stack memory. In the absence of recursion, an upper-bound on the use of stack memory can derived statically, thus making it possible to prove that an application will always live within its pre-allocated memory means.

### 4. Keep functions short 🤏🏻

**Rule**: No function should be longer than 80 lines of code.

**Rationale**: Each function should be a logical unit in the code that is understandable and verifiable as a unit. It is much harder to understand a logical unit that spans multiple screens on a computer display or multiple pages when printed. Excessively long functions are often a sign of poorly structured code.

### 5. Just blow up if you're stupid 💣

**Rule**: The assertion density of the code should average to a minimum of *two assertions per function*. Assertions are used to check for anomalous conditions that should never happen in real-life executions. Assertions must always be side-effect free and should be defined as Boolean tests. When an assertion fails - it means that you've made something so stupid that the program just have to *blow up*. Any assertion for which a static checking tool can prove that it can never fail or never hold violates this rule.

**Rationale**: Statistics for industrial coding efforts indicate that unit tests often find at least one defect per 10 to 100 lines of code written. The odds of intercepting defects increase with assertion density. Use of assertions is often also recommended as part of a strong defensive coding strategy. Assertions can be used to verify pre- and post- conditions of functions, parameter values, return values of functions, and loop-invariants. Because assertions are side-effect free, they can be selectively disabled after testing in performance-critical code.

### 6. Make data objects declaration scope as small as possible 💊

**Rule**: Data objects must be declared at the smallest possible level of scope.

**Rationale**: This rule supports a basic principle of data-hiding. Clearly if an object is not in scope, its value cannot be referenced or corrupted. Similarly, if an erroneous value of an object has to be diagnosed, the fewer the number of statements where the value could have been assigned; the easier it is to diagnose the problem. The rule discourages the re-use of variables for multiple, incompatible purposes, which can complicate fault diagnosis.

### 7. Check your data 🔍

**Rule**: The return value of non-void functions must be checked by each calling
function, and the validity of parameters must be checked inside each function.

**Rationale**: In its strictest form, this rule means that even the return value of `printf` statements and file close statements must be checked. One can make a case, though, that if the response to an error would rightfully be no different than the response to success, there is little point in explicitly checking a return value. This is often the case with calls to `printf` and `close`. In cases like these, it can be acceptable to explicitly cast the function return value to (void) – thereby indicating that the programmer explicitly and not accidentally decides to ignore a return value. In more dubious cases, a comment should be present to explain why a return value is irrelevant. In most cases the return value of a function should not be ignored, especially if error return values must be propagated up the function call chain. Standard libraries famously violate this rule with potentially grave consequences. By keeping the general rule, we make sure that exceptions must be justified, with mechanical checkers flagging violations. Often, it will be easier to comply with the rule than to explain why non-compliance might be acceptable.

### 8. Limit the use of preprocessor features ✋🏿

**Rule**: The use of the preprocessor must be limited to the inclusion of header files and simple macro definitions. Token pasting, variable argument lists (ellipses), and recursive macro calls are *not allowed*. All macros must expand into complete syntactic units. The use of conditional compilation directives is often also dubious, but cannot always be avoided. This means that there should rarely be justification for more than one or two conditional compilation directives even in large software development efforts, beyond the standard boilerplate that avoids multiple inclusion of the same header file. Each such use should be flagged by a tool-based checker and justified in the code.

**Rationale**: The effect of constructs in unrestricted preprocessor code can be extremely hard to decipher, even with a formal language definition in hand. Note that with just ten conditional compilation directives, there could be up to 2^10 possible versions of the code, each of which would have to be tested – causing a huge increase in the required test effort.

### 9. Restrict the use of pointers 👉🏿

**Rule**: The use of pointers should be restricted. Specifically, no more than one level of dereferencing is allowed. Pointer dereference operations may not be hidden in macro definitions or inside typedef declarations. Function pointers are *not permitted*.

**Rationale**: Pointers can make it hard to follow or analyze the flow of data in a program, especially by tool-based static analyzers. Function pointers, similarly, can seriously restrict the types of checks that can be performed by static analyzers and should only be used if there is a strong justification for their use, and ideally alternate means are provided to assist tool-based checkers determine flow of control and function call hierarchies. For instance, if function pointers are used, it can become impossible for a tool to prove absence of recursion, so alternate guarantees would have to be provided to make up for this loss in analytical capabilities.

### 10. Zero warnings ⚠️

**Rule**: All code must be compiled, from the first day of development, with all
compiler warnings enabled at the compiler’s most pedantic setting. All code must compile with these setting without any warnings. All code must be checked daily with at least one, but preferably more than one, state-of-the-art static source code analyzer and should pass the analyses with zero warnings.

**Rationale**: There is no excuse for any software development effort not to make use of static code analyzers. It should be considered routine practice. The rule of zero warnings applies even in cases where the compiler or the static analyzer gives an erroneous warning: if the compiler or the static analyzer gets confused, the code causing the confusion should be rewritten so that it becomes more trivially valid. Many developers have been caught in the assumption that a warning was surely invalid, only to realize much later that the message was in fact valid for less obvious reasons.

## List of references

* [NASA C Style Guide](https://ntrs.nasa.gov/api/citations/19950022400/downloads/19950022400.pdf)
* [The Power of Ten – Rules for Developing Safety Critical Code](https://spinroot.com/gerard/pdf/P10.pdf)
