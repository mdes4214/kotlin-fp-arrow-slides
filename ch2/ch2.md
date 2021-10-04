---

theme : "night"
transition: "slide"
highlightTheme: "monokai"
slideNumber: true
title: "Kotlin FP with Arrow"

---

### 2. Domain Modeling
### with ADT

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

### References

- http://www.natpryce.com/articles/000818.html
- https://agrawalsuneet.github.io/blogs/difference-between-any-unit-and-nothing-kotlin/

--

### TODO #1

```
- Top Type: Any
    - The root of the Kotlin class hierarchy. Every Kotlin class has Any as a superclass.
- Bottom Type: Nothing
    - Nothing is used to represent a value which will never exist.
    - Functions throwing exception directly
- Unit
    - Unit is exactly equivalent to the void type in Java.
    - Functions that return Unit can indicate a side effect inside.
```

--

### TODO #2

```
- Readability and Type Safety
- Inline Class
- Type Alias
- ADT (Algebraic Data Type)
- Product Type: data class
- Sum Type: enum class, sealed class
```

---

### Kotlin Type Hierarchy

![](img/hierarchy.png)

--

- Top Type - `Any`
- Bottom Type - `Nothing`
- `Unit`

---

### Any

<font size="6">The root of the Kotlin class hierarchy. 

➡️ Every Kotlin class has `Any` as a superclass.</font>

--

```kotlin=
fun main() {
  val num: Int = 5
  val anyVal: Any = num
  
  println("$anyVal") // 5
  println("${num.plus(10)}") // 15
  println("${anyVal.plus(10)}") // Error: Unresolved reference
}
```

--

`Any` is *similar* to the `Object` class in Java

➡️ `Any` only has 3 functions

```kotlin=
fun main() {
  val anyVal: Any = Any()
  val anyVal2: Any = Any()
  val objVal: Object = Object()

  println("${anyVal::class}") // class java.lang.Object
  println("${objVal::class}") // class java.lang.Object
    
  println("${anyVal.equals(anyVal2)}")  // false
  println("${anyVal == anyVal2}")       // false
  // A.equals(B) is the same as (A == B) if A is a non-null type
  
  println("${anyVal.hashCode()}") // 87285178
  
  println("${anyVal.toString()}") // java.lang.Object@533ddba
  println("$anyVal")              // java.lang.Object@533ddba  
}
```

---

### Nothing

<font size="6">`Nothing` is used to represent a value which will *never exist*.</font>

--

What kinds of expression evaluate to `Nothing`?

- ➡️ Those that perform control flow.
- ➡️ `throw` an exception
- ➡️ To represent the *absurd scenarios*.

--

```kotlin=
public inline fun TODO(): Nothing = throw NotImplementedError()

fun doOps(op: String): String = when (op) {
  "sum" -> "Sum up the values..."
  "square" -> TODO() // Note the type inference is not broken
  else -> "Do nothing."
}

fun main() {
  val result: String = doOps("square") // kotlin.NotImplementedError: An operation is not implemented.
  println("$result") // unreachable code
}
```

---

### Unit

<font size="6">The type with only one value: the `Unit` object.

🔍 This type corresponds to the `void` type in Java.</font>

--

Use `Unit` as return type of a function

- ➡️ Indicate a *Side Effect* !
- ➡️ The input must be *consumed* in the function

--

```kotlin=
import java.io.FileNotFoundException

fun printException(e: Exception): Unit = when (e) {
  is IllegalArgumentException -> println("The input is invalid: $e")
  is FileNotFoundException -> println("Cannot find the file: $e")
  else -> println("Other exception: $e")
}

fun main() {
  val e = IllegalArgumentException()
  printException(e) // Side Effect => the input "e" is consumed
}
```

---

### Why Type System

- Leverage compiler
  - ➡️ Type inference, Type check, ...
  - ➡️ Validation
- Code Readability
- One of the FP goals is to *bring certainty to the compiler with the types*
  - Pattern Matching
  - Domain Modeling

---

### Type Safety

---

### Inline Class

--

### Type Alias

---

### Algebraic Data Type (ADT)

- Product Type ➡️ `data class`
- Sum Type ➡️ `enum`, `sealed class`

---

### Product Type

---

### Sum Type