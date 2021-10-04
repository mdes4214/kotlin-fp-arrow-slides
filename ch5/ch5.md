---

theme : "night"
transition: "slide"
highlightTheme: "monokai"
slideNumber: true
title: "Kotlin FP with Arrow"

---

### 5. Kotlin Coroutine
### &
### Parallelization

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
- suspend
- Continuation
- either {}
- runBlocking {}
- coroutineScope {}: launch, async/await
- withContext()
```

--

### TODO #2

```
- Parallelization
    - parZip
    - parTraverse, parTraverseEither, parTraverseValidated
    - parSequence
    - raceN
- Schedule
    - repeat
    - retry
```
---

