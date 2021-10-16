---

theme : "night"
transition: "slide"
highlightTheme: "monokai"
slideNumber: true
title: "Kotlin FP with Arrow"

---

#### ch 3.
### Immutable Data Operation
### with Optics

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

- https://ivanmorgillo.com/2020/11/19/doh-there-is-no-copy-method-for-sealed-classes-in-kotlin/

---

### Immutability 

- Deterministic ➡️ same input, same output
- Pure Function ➡️ work over *immutable data*
- Easier to **test**
- Easier to **trace**
- Thread safety

---

### Data Class

`copy`

```kotlin=
// 展示copy是吐回另一個data class而不是修改
```

--

### Nested Data Class

Deep `copy`

```kotlin=
// 展示deep copy in data class
```

--

### 🔍 with Optics

```koltin=
// 展示@optics in deep copy
```

--

### Copy Sealed Class

- There is NO `copy` method for `sealed class`

--

### 🔍 with Optics

```koltin=
// 展示@optics prism for sealed class update
```

---

### Optics

- provided by [Arrow-kt](https://arrow-kt.io/docs/optics/)
- DSL (Domain-Specific Language)
	- ➡️ improve *ease of use* and *readability*
- **Generated at compile time**
- Direct and reusable syntax to read / modify immutable data structures
	- ➡️ "modify" means copy and *return a new instance*

--

### Arrow Optics

https://arrow-kt.io/docs/optics/
![](img/lens.png)

--

🔍 `Focus` means the target object
![](img/focus.png)

--

### Optics

- Lens
- Prism
- Optional
- Traversal
- Every
- Iso

---

### Lens

- Functional reference
- Focus into a structure and operate its focus
	- `get` ➡️ get the focus
	- `set` ➡️ set the focus to input value
	- `modify` ➡️ update the focus with input function

--

```kotlin=
// sample for Lens
```
	
--

### Prism

- See into a structure and optionally find its focus
	- `getOrModify` ➡️ match the focus 
	- `reverseGet` ➡️ get back to the source domain from the focus
- Mostly used for structures that have a relationship only under a *certain condition*
	- ➡️ Copy method for `sealed class` !

--

```kotlin=
// sample for Prism
```	
	
--

### Optional

- Allow seeing into a structure and *getting, setting, or modifying an optional focus*
	- Lens ➡️ getting, setting, or modifying
	- Prism ➡️ an optional focus

--

🔍 `Optional` is composed of `Lens` and `Prism`

```kotlin=
// sample for compose Optional
```	
	
--

```kotlin=
// sample for Optional
```	
	
---

### Traversal

- See into a structure and set, or modify 0 to N foci
- Focus into a structure that has 0 to N elements
	- such as *collections* etc.

--

```kotlin=
// sample for simple list Traversal
```

--

```kotlin=
// sample for compose Traversal
```

--

🔍 Leverage DSL

```kotlin=
// sample for Traversal DSL
```
	
--

### Every

- See into a structure and set, or modify 0 to N foci
- Focus into a structure that has 0 to N elements
	- such as *collections* etc.
- **Match over elements** on a collection

--

```kotlin=
// sample for compose Every
```

--

🔍 Leverage DSL
```kotlin=
// sample for Every DSL
```

---

### Iso

- Define an **isomorphism** between a type `S` and `A`
	- `data class` and `TupleN`
- **Invertible** optic

--

```kotlin=
// sample for Iso
```

--

🔍 Compose with other optics
```kotlin=
// sample for compose Iso with other optics
```

---

### Recap #1

- Immutability
	- Deep copy in nested `data class`
	- NO copy method for `sealed class`
- `@Optics`
  - Solved above problems
  - DSL for highly *composable* and *reusable*
  - **Generated at compile time**

--

### Recap #2

- Lens, Prism ➡️ compose to Optional
- Traversl, Every ➡️ for collections
- Iso ➡️ isomorphism between `data class` and `TupleN`



