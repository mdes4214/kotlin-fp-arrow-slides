---

theme : "night"
transition: "slide"
highlightTheme: "monokai"
slideNumber: true
title: "Kotlin FP with Arrow"

---

#### ch 8. 
### Functional Architecture

<style>
pre {
  background: #303030;
  padding: 10px 16px;
  border-radius: 0.3em;
  counter-reset: line;
}
pre code[class*="="] .line {
  display: block;
  line-height: 1.8rem;
  font-size: 1em;
}
pre code[class*="="] .line:before {
  counter-increment: line;
  content: counter(line);
  display: inline-block;
  border-right: 3px solid #6ce26c !important;
  padding: 0 .5em;
  margin-right: .5em;
  color: #afafaf !important;
  width: 24px;
  text-align: right;
}

.reveal .slides > section > section {
  text-align: center; 
}

h1,h2,h3,h4 {
  text-align: center;
}

p {
  text-align: center;
}
</style>

--

[return to Outline](../../export/#/2)

--

### References #1

- https://book.kotlincn.net/
- https://kotest.io/

--

### References #2

- https://mjmanaog.medium.com/kotlin-abstract-class-interface-b9c4caf22252
- https://softwareengineering.stackexchange.com/questions/275891/is-functional-programming-a-viable-alternative-to-dependency-injection-patterns

---

### Runtime & Program Logic

(圖示意runtime(How to consume) <-> program description(What it does))

--

### Runtime

🔍 interpreter of the program

- Decide how to consume the program
  - ➡️ pick the execution strategy
- Provide dependencies to the program
  - ➡️ implementation detail

--

### Program Logic

🔍  agnostic of the runtime

- Program logic can remain pure and abstract
  - ➡️ lift the function
  - ➡️ `suspend`
- Pure edge to edge
  - ➡️ **input -> output**
  
--

### How about Side Effect ?

➡️ `suspend`

- Lift the function and defer side effects to runtime
  - the entry point requires providing a runtime (*Coroutine*) to consume
- Allows keeping side effects tracked by compiler
  - can't be called from everywhere
- Pure Architecture
  - push side effects to the **edges**
  
--

#### Push Side Effects to the Edges

(圖，包含entry point, pure architecture, side effects, 左右都是the edge of the world，先不要有dependencies)

🔍 Note the side effects are flagged with `suspend`

---

### Logger in FP

How to do logging in Functional Programming ?
- [Reference 1](https://www.raywenderlich.com/9527-functional-programming-with-kotlin-and-arrow-getting-started#toc-anchor-018)
- [Reference 2](https://darrmirr.medium.com/logging-at-functional-programming-efd95da734d1)
- [Reference 3](https://www.reddit.com/r/fsharp/comments/5d8ivi/logging_in_f_or_any_other_functional_language/)

--

### [Imperative vs. Declarative](https://medium.com/disney-streaming/option-either-state-and-io-imperative-programming-in-a-functional-world-8e176049af81)

The fundamental dichotomy between *imperative* and *functional (declarative)* programming

➡️ statements vs. expressions

--

### Statements vs. Expressions

- Statements
  - commands to be executed that *interact with*, and possibly *alter, the world*
- Expressions
  - *pure* computations that are evaluated to yield a result
  - think *arithmetic expressions* as the canonical example
  
--

### How is I/O ?

I/O operation is a statement with *side effects*, and neither composable nor reusable.
- read/write files
- interact with database
- standard input and output
- **logging**

--

### About `IO` Type

In functional programming, there is a `IO` type
- let the statement return a value of `IO<T>`
- lift the function ➡️ defer side effects

```kotlin=
// sample code for IO
```

--

### About `IO` Type

- **Treat statements as expressions**
- Pure & Reusable

➡️ [In Kotlin, that is `suspend`.](https://jorgecastillo.dev/please-try-to-use-io)

--

- Flagging a side effect as `suspend` effectively makes it **pure**
- The value you're returning is a description of an effect
- ➡️ not performing the effect itself unless you provide the **mentioned environment**

--

```kotlin=
// sample code 跟IO一樣，但是換成suspend
```

--

### Sample of Logger

Leverage `suspend`

```kotlin=
// 配合上面的reference，sample code寫一個logger class, info那些寫suspend fun 回Either<Error, T>，T用泛型，等於可以composable & reusable
```

---

### Algebras

- Pure function as **Input -> Output**
  - ➡️ algebraic equation
- Referential Transparency
  - ➡️ compose pure functions
  - ➡️ reduce the program to value
  - ➡️ just like the functions in mathematics

--

### Algebras

- If a functional program is "pure functions + immutable data"
  - ➡️ deterministic algebraic equations
- Pure functions ➡️ Operators (**operations**)
- Immutable data ➡️ Values (**objects**)
- Laws to ensure properties of the algebra
  - Arrow-kt guarantees integrity and correctness

--

### Algebras

🔍 comprised by a set of **objects** and the **operations** to work with those

⬇️

- ADTs + Interpreter
- Interface

--

### ADTs + Interpreter

Interpreter evaluates the operation and provides implement details

```kotlin=
// sample for ADTs + Interpreter algebras
```

--

### Interface

```kotlin=
// sample for interface algebras
```

🤔 How to implement and "pass" to our program ?

---

### [Dependency Injection](https://softwareengineering.stackexchange.com/questions/275891/is-functional-programming-a-viable-alternative-to-dependency-injection-patterns)

Why dependency management is a big problem in OOP?
- The tight coupling of data and code
- Ubiquitous use of side effects

--

🤔 Does we really need Dependency Injection

in Functional Programming ?

--

In functional world, 

if you think in terms of **inputs and outputs** ...

➡️ There is *no function dependency, only data dependency*.

🔍 It's just a composition of (A) -> B, (B) -> C, etc.

--

➡️ That's why FP is easy to test.

--

### With Side Effect

In real world, 

we usually need to deal with side effects

- ➡️ Sometimes need to change implementation
- ➡️ Still need DI
- ➡️ But we can do it gracefully, make it *testable*

--

#### Push Side Effects to the Edges

(圖，包含entry point, pure architecture, side effects, 左右都是the edge of the world，這次要有dependencies)

- Target abstractions, provide side effects as **implementation details**
  - **Inject** the implementation details

--

### Dependency Injection in FP

- Explicit
  - ➡️ pass dependencies as function arguments
  - ➡️ **pollute** syntax
- Implicit
  - ➡️ leverage mechanisms to pass dependencies implicitly
  - ➡️ only initial caller provides them
  - ➡️ access them only where needed

---

### Implicit DI

➡️ via Extension Function

- Work over a **scope**
- The other required scopes are provided by initial caller
- Can access to all properties of the scope

--

```kotlin=
// sample for 一系列dependencies extension function
```

--

Only pass dependencies on the first call.

```kotlin=
// sample for Dependencies initialize and pass other dependency scopes as arguments
```

🔍 Extension functions are resolved statically, hence this approach is compile time checked.

--

### Implicit DI

Any program with effects can be described as
1. `suspend (Dependencies) -> Result`
2. `suspend Dependencies.() -> Result`

```kotlin=
// sample for 上面兩者的比較，註明第二種是implicit
```

--

#### A nice approach
- pass dependencies *implicitly* via extension functions
- leave *explicit* parameters for the payload of our functions

--

### Further reading - [Receiver](https://kotlinlang.org/docs/lambdas.html#function-literals-with-receiver)

`A.() -> B`

➡️ Used for [DSL](https://stackoverflow.com/questions/45875491/what-is-a-receiver-in-kotlin) and... remember the Scope Function ?

```kotlin=
// apply{}, let{}的定義
```

---

### Dependency Scoping

<font size="6">We can segregate scopes and compose in different ways depending on the use cases we need to model.</font>

(圖，兩層scope就好，配合後面的sample code)

--

### Dependency Scoping

```kotlin=
// sample for 上圖的兩層scope
```

---

### [Delegation Pattern](https://en.wikipedia.org/wiki/Delegation_pattern)

```kotlin=
// sample from wiki, 標示出delegate(代表), delegator(委託人), delegation(委託)，加上main
```

--

### [Delegation in Kotlin](https://kotlinlang.org/docs/delegation.html)

`by`

```kotlin=
// sample from wiki, 換成by，一樣標出那三個，一樣有main
```

--

### [Delegation Properties](https://kotlinlang.org/docs/delegated-properties.html)

`lazy` ➡️ the value is computed only on first access

- First call to `get()`
  - execute the lambda passed to lazy()
  - remember the result
- Subsequent calls to `get()`
  - simply return the remembered result

--

```kotlin=
// sample for lazy
```

---

### Instance of Abstract Class

```kotlin=
// very simple abstract class to show abstract class cannot have a instance
```

--

### Object of Anonymous Class

```kotlin=
// sample for object expression
```

➡️ Singleton

➡️ [Object Expression](https://kotlinlang.org/docs/object-declarations.html)

---

### Abstract Class vs. Interface

Basically, an abstract can do what an interface can

➡️ both subclass / implementation need to implement abstract function

🤔 What is the difference?

--

### Abstract Class

can have ...

- **constructors**
- an **init body**
- properties with given **value**

--

```kotlin=
// sample for simple abstract class with constructors, init body, properties with value. 
```

--

➡️ Delegation for lazy initialization

```kotlin=
// sample for abstract class dependency scope (same as before)
```

--

### Interface

You only want to share behavior with the class but not the code between a set of objects

```kotlin=
// sample for simple interface
```

--

➡️ Set of operations for algebras

```kotlin=
// sample for algebras (same as before)
```

🔍 Program to an interface, not an implementation.

---

### Dependencies in Testing

Recap the Dependency Scoping

```kotlin=
// sample for scoping (same as before)
```

--

Define scopes as interfaces or abstract classes for flexibility

➡️ replace implementation in tests

```kotlin=
// sample for 加入scope當參數的實作class
```

--

With algebras, we can override the dependencies we want to replace only

```kotlin=
// sample for test algebras
```

➡️ Mock the dependencies and test without touching the real network or database

--

Alternatively, use [Testcontainers](https://www.testcontainers.org/)!

--

### Implications of FP for testing

- Purity
  - <font size="6">deterministic, predictable, no side effect</font>
  - <font size="6">➡️ **Remove flakiness**</font>
- Immutability
  - <font size="6">no unexpected state changes</font>
  - <font size="6">➡️ Test scenarios and assertions **remain correct**</font>

--

### Implications of FP for testing

- Functional domain modeling
  - <font size="6">strict data validation (types + compiler)</font>
  - <font size="6">➡️ Remove the need of tests for **invariants**</font>
- And more
  - <font size="6">error handling, ADT, implicit DI</font>
  - <font size="6">➡️ **Testability**, **Exhaustivity**</font>

---

### Recap #1

- Pure Logic
  - describe *what to do*
  - push Side Effects to the **Edges**
  - implement at runtime (*how to consume*)
  - `IO` and `suspend` and logger
- Algebras
  - ADTs + interpreter
  - interface

--

### Recap #2

- Dependency Injection
  - via Extension Function
  - pass dependencies **implicitly**
  - leave explicit arguments for payload
- Dependency Scoping
  - delegation, object expression
  - *program to an interface, not an implementation*
  - replace implementation in tests


