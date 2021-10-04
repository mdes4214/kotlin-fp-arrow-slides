---

theme : "night"
transition: "slide"
highlightTheme: "monokai"
slideNumber: true
title: "Kotlin FP with Arrow"

---

### 4. Error Handling
### with Either & Validated

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

- 

--

### TODO #1

```
- Either<L, R>
- Evaluation: fold
- Composition
    - flatMap
    - either.eager {} with bind
    - zip
- 3rd Party Error: Either.catch, mapLeft
- Short circuit on error
- Flatten
    - traverseEither 
    - Collection<Either<E, A>> => Either<E, Collection<A>>
- fromNullable
```

--

### TODO #2

```
- Validated
- ValidatedNel
- zip
- Accumulate invalid cases
- Flatten
    - traverseValidated 
    - Collection<ValidatedNel<E, A>> => ValidatedNel<E, Collection<A>>
- fromNullable
```

---

