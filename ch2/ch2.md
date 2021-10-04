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

