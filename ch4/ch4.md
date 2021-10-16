---

theme : "night"
transition: "slide"
highlightTheme: "monokai"
slideNumber: true
title: "Kotlin FP with Arrow"

---

#### ch 4.
### Domain Modeling 
#### and 
### Error Handling

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

- https://fsharpforfunandprofit.com/posts/recipe-part2
- https://fsharpforfunandprofit.com/rop/
- https://hltj.me/kotlin/2017/08/25/kotlin-functor-applicative-monad-cn.html

---

### Domain Modeling Recap

- Why Type System
	- Communicate the design in the code
	- Type Safety
- 🔍 **Inline Class**
- ADT ➡️ **Product Type**, **Sum Type**

--

[Domain Modeling Made Functional - Scott Wlaschin](https://youtu.be/2JB1_e5wZmU)
<img height="400" src="img/scott_wlaschin.png">

---

### Error Handling

- `Either` ➡️ model error and success
- `Validated` ➡️ accumulate errors
- `Option` ➡️ for absent optional value
	- Null-safety mechanism in Kotlin
- provided by [Arrow-kt](https://arrow-kt.io/docs/core/)

--

### Arrow Core
https://arrow-kt.io/docs/core/
![](img/either.png)

---

### Railway Oriented Programming

<img height="400" src="img/rop.png">

--

input ➡️ Success / Failure output

<img height="300" src="img/rop_2.png">

---

### Either

- `Left` side and `Right` side
- Implement by `sealed class`

➡️ Errors on `Left` side as convention

```kotlin=
// Either sealed class
// `Nothing` used for the "non-relevant" side
```

--

### Exception ?

- What kind of exceptions a function may throw
	- ➡️ Should dig through the source code
- How to handle exceptions
	- ➡️ Should make sure catch them all

**⚠️ Unwieldy !**

--

### 🔍 with Either

- Functions return `Either`
  - ➡️ explicit about all scenarios
- More exhaustive
  - ➡️ enforce callers to treat both sides

```kotlin=
// Sample for Either (left和right都要)
```

--

### Evaluation Either Results

➡️ `fold`

```kotlin=
// sample for fold
```

--

### Either Composition

<img width="600" src="img/rop_3.png">

⬇️

<img width="600" src="img/rop_4.png">

--

<img width="600" src="img/rop_5.png">

⬇️

<img width="600" src="img/rop_6.png">

--

### flatMap

- Chain `Either` sequentially
- A kind of the *M*-word ➡️ **`Monad`**

```kotlin=
// sample for flatMap (串兩三個Either)
```

--

<img height="600" src="img/monad.png">

--

### flatMap

- Compute over the happy case ➡️ Happy Path
- Short circuit on error ➡️ Failure Path

<img width="600" src="img/rop_6.png">

--

```kotlin=
// sample for short circuit
```

--

### Computation Block

➡️ Translate to chain of `flatMap` at compile time

```kotlin=
// sample for flatMap (串兩三個Either) => 改寫成either.eager{}
```

--

### 3rd Party Error

🔍 Sometimes you do need to interact with code that can potentially **throw exceptions**

➡️ `Either.catch {} .mapLeft {}`

--

🔍 Define a `toDomain` method to map `Throwable` to **domain error**

```kotlin=
// sample for Either.catch {} (注意e.toDomain)
```

---

### Error Accumulation

Sometimes it would be nice to **have all of these errors** reported simultaneously.

➡️ e.g. validate the input form

--

![](img/rop_7.png)

--

### Validated

- `Invalid` side and `Valid` side

```kotlin=
// sample for Validated implement sealed class
```

--

**Accumulate** errors in `Invalid` side

➡️ `ValidatedNel`

--

### ValidatedNel

- Nel ➡️ `NonEmptyList`
- Accumulate invalid cases

🔍 `NonEmptyList` is a data type used in Arrow to model ordered lists that guarantee to have at least one value.

--

### ValidatedNel

```kotlin=
// sample for ValidatedNel implement sealed class
```

--

### Define Validate Functions

```kotlin=
// sample for ValidateNel functions (可以用檢查表單欄位當例子)
```

--

### Zip to Accumulate

```kotlin=
// sample for zip in ValidateNel
```

--

### Validate Domain Model Creation

```kotlin=
// Sample for validate when domain model create
```

---

### Either vs. Validated

- `Either` ➡️ Short circuit on error
- `Validated`, `ValidatedNel` ➡️ Accumulate invalid cases

--

### Either & Validated

```kotlin=
// sample for Either & Validated (用either.eager{}包起來，檢查表單欄位後送出)
```

---

### Option

- `Option<A>`
	- if value `A` is present ➡️ `Some<A>`
	- if value is absent ➡️ `None`

--

### Option

- Can be *emulated* by `Either<Unit, A>` and nullable variable `A?`
	- https://github.com/arrow-kt/arrow-core/issues/114

--

As a FP idiom and to support some use cases (RxJava) 

➡️ use `Option`

--

`fromNullable` and `A?.let {}`

```kotlin=
// sample for Option, Either<Unit, A>, A? (A?.let 模擬flatMap)
```

---

### Flatten Iterable Collection

- Flatten **iterable** collection to make it easier to work with
	- `Array<T>` doesn't implement `Iterable<T>`

--

- `List<Either<E, A>>`
  - ➡️ `Either<E, List<A>>`
- `List<ValidateNel<E, A>>`
  - ➡️ `ValidateNel<E, List<A>>`
- `List<Option<A>>`
  - ➡️ `Option<List<A>>`

--

- `sequence` ➡️ simply flatten
- `traverse` ➡️ flatten after *operation*

```kotlin=
// sample for Flatten iterable (用either.eager{}串起這些flatten後的)
```

---

### Recap #1

- Railway Oriented Programming
  - ➡️ Success / Failure output
- `Either<E, A>`
  - Evaluation ➡️ `fold`
  - Composition ➡️ `flatMap`
    - ➡️ `either.eager {}` with `bind()`
  - Handling 3rd Party Error
    - ➡️ `Either.catch {} .mapLeft {}`
  - Short circuit on error

--

### Recap #2

- `Validated<E, A>` & `ValidatedNel<E, A>`
	- Accumulate invalid cases ➡️ `zip`	
- `Option<A>`
	- Absent optional value ➡️ `Some<A>`, `None`
	- `fromNullable` and `A?.let {}`
- Flatten
  - `sequence`, `traverse`
  - `Iterable<Either<E, A>>`
    - ➡️ `Either<E, Iterable<A>>`