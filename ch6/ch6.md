---

theme : "night"
transition: "slide"
highlightTheme: "monokai"
slideNumber: true
title: "Kotlin FP with Arrow"

---

#### ch 6.
### Safe Resource Handling

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

- https://kotlin.github.io/kotlinx.coroutines/index.html

---

### What is a resource ?

- Database
- File (IO)
- Connection
- etc.

--

### What is safe handling ?

- Thread safety
- Unchanged data state
- No risk of concurrent modification
- No need of synchronization
- No risk of deadlocks

➡️ leverage Immutability

--

### Immutable Data

- `val` ➡️ still mutable when it points to a mutable data structure
  - `val numbers = mutableListOf(1, 2, 3)`
- Immutable collections ➡️ read-only
  - `val numbers = listOf(1, 2, 3)`
- 3rd party provided data structure for safe-accessed concurrency

--

### 3rd Party Libraries

- Arrow-kt
  - `Atomic<A>` ➡️ Concurrent safe Reference
- Kotlinx
  - `Mutex` ➡️ Mutual exclusion for coroutines
  - `Semaphore` ➡️ Counting semaphore for coroutines
  - `Channel` ➡️ Communication between coroutines
    - <font size="6">similar to `BlockingQueue` in Java, but with **suspending** operations</font>
  
--

### Atomic

- provided by [Arrow-kt](https://arrow-kt.io/docs/apidocs/arrow-fx-coroutines/arrow.fx.coroutines/-atomic/)
- In other languages also known as `Ref`, `IORef`
  - ➡️ `suspend`
- A wrapper around Kotlinx [`AtomicFU`](https://github.com/Kotlin/kotlinx.atomicfu)

--

```kotlin=
import arrow.fx.coroutines.Atomic

suspend fun main() {
    val num = Atomic(15)

    println(num.access()) // Obtains a snapshot of the current value, and a setter for updating it
    val numSetter = num.access().second
    numSetter(20)
    println(num.get()) // 20
    num.set(15)

    println(num.get()) // 15
    println(num.getAndUpdate { it * 2 }) // 15, then update `num` to 15 * 2
    println(num.get()) // 30
    num.update { it * 2 }
    println(num.get()) // 60

    // Create an AtomicRef
    val numLens = num.lens(
        get = { it + 2 },
        set = { _, newValue -> newValue + 3 }
    )
    println(numLens.get()) // 60 + 2 = 62
    numLens.set(15) // 15 + 3 = 18
    println(numLens.get()) // 18 + 2 = 20
}
```

---

### Working with a resource

- Acquisition
- Using
- Releasing ➡️ always need to do

🔍 Asynchronous & Coroutines

🔍 Dependency Injection

--

### Try / Catch / Finally

- Problems with handling `cancel()` in **coroutines**
  - `catch` captures also `CancellationException`
  - `acquire` and `release` need to be `NonCancellable`
- Can become messy with segregated error handling

--

```kotlin=
import kotlinx.coroutines.CancellationException
import kotlinx.coroutines.NonCancellable
import kotlinx.coroutines.withContext
import other.model.SuspendFile

suspend fun main() {
    // file open, readContent, and close are suspended ops
    val file = withContext(NonCancellable) { // acquire need to be NonCancellable
        SuspendFile("FP_note.txt").open()
    }
    try {
        println(file.readContent())
    } catch (e: CancellationException) { // catch also CancellationException
        withContext(NonCancellable) { file.close() } // release need to be NonCancellable
    } catch (t: Throwable) {
        withContext(NonCancellable) { file.close() } // release need to be NonCancellable
    }
}
```

---

### Safe Resource Handling

- `bracket` & `bracketCase`
- `Resource`
- `guarantee` & `guaranteeCase`
- `onCancel`
- `CircuitBreaker`
- provided by [Arrow-kt](https://arrow-kt.io/docs/fx/)

--

### Arrow Fx

https://arrow-kt.io/docs/fx/
![](img/resource.png)

--

🔍 Acquire, use, and release resources *regardless of how and from where they are used*.

🔍 Supports any other Arrow Fx operators

➡️ `suspend` !

--

### Always release

A resource should be released in 3 cases
- Completion
- Cancellation
- Failure

---

### [bracket](https://arrow-kt.io/docs/apidocs/arrow-fx-coroutines/arrow.fx.coroutines/bracket.html)

Declare `acquire`, `use`, and `release`

```kotlin=
suspend fun <A, B> bracket(
  acquire: suspend () -> A,
  use: suspend (A) -> B,
  release: suspend (A) -> Unit
): B
```

--

### [bracketCase](https://arrow-kt.io/docs/apidocs/arrow-fx-coroutines/arrow.fx.coroutines/bracket-case.html)

Same as `bracket`, but `release` by [`ExitCase`](https://arrow-kt.io/docs/apidocs/arrow-fx-coroutines/arrow.fx.coroutines/-exit-case/index.html)

```kotlin=
sealed ExitCase {
  object Completed: ExitCase()
  data class Cancelled(val exception: CancellationException) : ExitCase()
  data class Failure(val failure: Throwable) : ExitCase()
}

suspend fun <A, B> bracketCase(
  acquire: suspend () -> A, 
  use: suspend (A) -> B, 
  release: suspend (A, ExitCase) -> Unit
): B
```

--

### Error Handling

- Release if there is an error while using the resource
- `bracket` and `bracketCase` will rethrow exceptions
  - ➡️ use `Either.catch {}`

--

```kotlin=
import arrow.core.Either
import arrow.fx.coroutines.ExitCase
import arrow.fx.coroutines.bracketCase
import other.model.SuspendFile

suspend fun main() {
    val result = Either.catch {
        bracketCase(
            acquire = { SuspendFile("FP_note.txt").open() },
            use = { throw RuntimeException("Boom!") },
            release = { file, exitCase ->
                when(exitCase) {
                    is ExitCase.Completed -> { println("Release with Completed") }
                    is ExitCase.Cancelled -> { println("Release with Cancelled") }
                    is ExitCase.Failure -> { println("Release with Failure") } // will run
                }
                file.close()
            }
        )
    }

    println(result) // Either.Left(java.lang.RuntimeException: Boom!)
}
```

--

### Cancellation Handling

Release when the coroutine is cancelled

```kotlin=
import arrow.fx.coroutines.ExitCase
import arrow.fx.coroutines.bracketCase
import arrow.fx.coroutines.never
import kotlinx.coroutines.async
import kotlinx.coroutines.coroutineScope
import other.model.SuspendFile

suspend fun main() {
    coroutineScope {
        val job = async {
            bracketCase(
                acquire = { SuspendFile("FP_note.txt").open() },
                use = { never<Unit>() },
                release = { file, exitCase ->
                    when(exitCase) {
                        is ExitCase.Completed -> { println("Release with Completed") }
                        is ExitCase.Cancelled -> { println("Release with Cancelled") } // will run
                        is ExitCase.Failure -> { println("Release with Failure") }
                    }
                    file.close()
                }
            )
        }

        job.cancel()
    }
        
    // File [FP_note.txt] opened
    // Release with Cancelled
    // File [FP_note.txt] closed
}
```

--

If you fail or cancel on `acquire`, then `release` is not called

➡️ Can't release something we never acquired

--

### Limitations

`bracket` and `bracketCase` tied to decide how to **use** the resource when we `acquire` it

➡️ Can we separate `acquire` and `use` ?

```kotlin=
suspend fun <A, B> bracket(
  acquire: suspend () -> A,
  use: suspend (A) -> B, // this makes it hardly reusable
  release: suspend (A) -> Unit
): B

suspend fun <A, B> bracketCase(
  acquire: suspend () -> A, 
  use: suspend (A) -> B, // this makes it hardly reusable
  release: suspend (A, ExitCase) -> Unit
): B
```

---

### [Resource](https://arrow-kt.io/docs/apidocs/arrow-fx-coroutines/arrow.fx.coroutines/-resource/index.html)

- Declare `acquire` and `release`
- It's a datatype
  - ➡️ we can pass around or inject and reuse
- Adhere to [Monad Laws](https://arrow-kt.io/docs/patterns/monads/index.html#monad-laws) like `Either`, `Option`
  - ➡️ `map`, `flatMap`, `traverse`, ...
  - ➡️ compose with other `suspend` operations
  
--

```kotlin=
import arrow.fx.coroutines.ExitCase
import arrow.fx.coroutines.Resource
import other.model.SuspendFile

suspend fun main() {
    val resource = Resource(
        acquire = { SuspendFile("FP_note.txt").open() },
        release = { file, exitCase ->
            when(exitCase) {
                is ExitCase.Completed -> { println("Release with Completed") }
                is ExitCase.Cancelled -> { println("Release with Cancelled") }
                is ExitCase.Failure -> { println("Release with Failure") }
            }
            file.close()
        }
    )

    // reuse the Resource
    resource.use { file -> println(file.readContent()) }
    println("---")
    resource.use { file -> println("Just want to print the file name: ${file.fileName}") }

    // File [FP_note.txt] opened
    // The content of [FP_note.txt]
    // Release with Completed
    // File [FP_note.txt] closed
    // ---
    // File [FP_note.txt] opened
    // Just want to print the file name: FP_note.txt
    // Release with Completed
    // File [FP_note.txt] closed
}
```

--

<font size="6">⚠️ Can be inefficient to **acquire and release every time** if we have many operations to perform over the same resource</font>

```kotlin=
import arrow.fx.coroutines.ExitCase
import arrow.fx.coroutines.Resource
import other.model.SuspendFile

suspend fun main() {
    val resource = Resource(
        acquire = { SuspendFile("FP_note.txt").open() },
        release = { file, exitCase ->
            when(exitCase) {
                is ExitCase.Completed -> { println("Release with Completed") }
                is ExitCase.Cancelled -> { println("Release with Cancelled") }
                is ExitCase.Failure -> { println("Release with Failure") }
            }
            file.close()
        }
    )
    
    for (idx in 1..5) {
        // acquire and release every time
        resource.use { file -> println("Use the same Resource($idx/5): ${file.readContent()}") }
        println("---")
    }
}
```

--

### Solve the Inefficiency

- Fit everything inside the `use` lambda
- Leverage architecture design
  - ➡️ reuse connection from a pool
- Push the problem into implementation details
  - ➡️ we can still rely on the *abstractions*

--

### Compose Resource

<font size="6">🔍 Resources guarantee that their release finalizers are always invoked in the correct **order** when `Cancelled` and `Failure`</font>

```kotlin=
import arrow.core.Either
import arrow.fx.coroutines.ExitCase
import arrow.fx.coroutines.Resource
import other.model.SuspendFile

suspend fun printExitCaseThenClose(file: SuspendFile, exitCase: ExitCase) {
    when(exitCase) {
        is ExitCase.Completed -> { println("Release with Completed") }
        is ExitCase.Cancelled -> { println("Release with Cancelled") }
        is ExitCase.Failure -> { println("Release with Failure") }
    }
    file.close()
}

suspend fun main() {
    val resources = Resource(
        acquire = { SuspendFile("FP_note.txt").open() },
        release = { file, exitCase -> printExitCaseThenClose(file, exitCase) }
    ).zip(
        Resource(
            acquire = { SuspendFile("Domain Modeling Made Functional.pdf").open() },
            release = { file, exitCase -> printExitCaseThenClose(file, exitCase) }
        ),
        Resource(
            acquire = { SuspendFile("end of a life.mp3").open() },
            release = { file, exitCase -> printExitCaseThenClose(file, exitCase) }
        )
    ) { file1, file2, file3 ->
        println("Zip Resources [${file1.fileName}], [${file2.fileName}], [${file3.fileName}]")
        Triple(file1, file2, file3)
    }

    Either.catch {
        resources.use { files -> throw RuntimeException("Boom!") } // guarantee the release order
    }

    // Release with Failure
    // File [end of a life.mp3] closed
    // Release with Failure
    // File [Domain Modeling Made Functional.pdf] closed
    // Release with Failure
    // File [FP_note.txt] closed
}
```

--

If you fail or cancel on `acquire`, then `release` is not called

➡️ Can't release something we never acquired

🔍 `acquire` & `release` step are `NonCancellable`
 
--

### Error Handling

- Release if there is an error while using the resource
- `use` will rethrow exceptions
  - ➡️ use `Either.catch {}`

--

```kotlin=
import arrow.core.Either
import arrow.fx.coroutines.ExitCase
import arrow.fx.coroutines.Resource
import other.model.SuspendFile

suspend fun main() {
    val resource = Resource(
        acquire = { SuspendFile("FP_note.txt").open() },
        release = { file, exitCase ->
            when(exitCase) {
                is ExitCase.Completed -> { println("Release with Completed") }
                is ExitCase.Cancelled -> { println("Release with Cancelled") }
                is ExitCase.Failure -> { println("Release with Failure") } // will run
            }
            file.close()
        }
    )

    val result = Either.catch {
        resource.use { throw RuntimeException("Boom!") }
    }

    // File [FP_note.txt] opened
    // Release with Failure
    // File [FP_note.txt] closed
}
```

--

### Cancellation Handling

Release when the coroutine is cancelled

```kotlin=
import arrow.fx.coroutines.ExitCase
import arrow.fx.coroutines.Resource
import arrow.fx.coroutines.never
import kotlinx.coroutines.async
import kotlinx.coroutines.coroutineScope
import other.model.SuspendFile

suspend fun main() {
    coroutineScope {
        val resource = Resource(
            acquire = { SuspendFile("FP_note.txt").open() },
            release = { file, exitCase ->
                when(exitCase) {
                    is ExitCase.Completed -> { println("Release with Completed") }
                    is ExitCase.Cancelled -> { println("Release with Cancelled") } // will run
                    is ExitCase.Failure -> { println("Release with Failure") }
                }
                file.close()
            }
        )

        val job = async { resource.use { never<Unit>() } }

        job.cancel()
    }

    // File [FP_note.txt] opened
    // Release with Cancelled
    // File [FP_note.txt] closed
}
```

---

### [guarantee](https://arrow-kt.io/docs/apidocs/arrow-fx-coroutines/arrow.fx.coroutines/guarantee.html)

- Guarantee execution of a given `finalizer` after `fa` regardless of success, error or cancellation
- Not meant for handling resources but any suspended effects

```kotlin=
suspend fun <A> guarantee(
  fa: suspend () -> A, 
  finalizer: suspend () -> Unit
): A
```

--

### [guaranteeCase](https://arrow-kt.io/docs/apidocs/arrow-fx-coroutines/arrow.fx.coroutines/guarantee-case.html)

Same as `guarantee`, but `finalizer` with [`ExitCase`](https://arrow-kt.io/docs/apidocs/arrow-fx-coroutines/arrow.fx.coroutines/)

```kotlin=
suspend fun <A> guaranteeCase(
  fa: suspend () -> A, 
  finalizer: suspend (ExitCase) -> Unit
): A
```

--

### Error Handling

- Run `finalizer` if there is an error while running the effect
- `guarantee` and `guaranteeCase` will rethrow exceptions after running `finalizer`
  - ➡️ use `Either.catch {}`

--

```kotlin=
import arrow.core.Either
import arrow.fx.coroutines.ExitCase
import arrow.fx.coroutines.guaranteeCase

suspend fun main() {
    val result = Either.catch {
        guaranteeCase(
            fa = { throw RuntimeException("Boom!") },
            finalizer = { exitCase ->
                when(exitCase) {
                    is ExitCase.Completed -> { println("Finalizer with Completed") }
                    is ExitCase.Cancelled -> { println("Finalizer with Cancelled") }
                    is ExitCase.Failure -> { println("Finalizer with Failure") } // will run
                }
            }
        )
    }

    println(result) // Either.Left(java.lang.RuntimeException: Boom!)
}
```

--

### Cancellation Handling

Run `finalizer` when the coroutine is cancelled

```kotlin=
import arrow.fx.coroutines.ExitCase
import arrow.fx.coroutines.guaranteeCase
import arrow.fx.coroutines.never
import kotlinx.coroutines.async
import kotlinx.coroutines.coroutineScope

suspend fun main() {
    coroutineScope {
        val job = async {
            guaranteeCase(
                fa = { never<Unit>() },
                finalizer = { exitCase ->
                    when(exitCase) {
                        is ExitCase.Completed -> { println("Finalizer with Completed") }
                        is ExitCase.Cancelled -> { println("Finalizer with Cancelled") } // will run
                        is ExitCase.Failure -> { println("Finalizer with Failure") }
                    }
                }
            )
        }

        job.cancel()
    }
}
```

--

### [onCancel](https://arrow-kt.io/docs/apidocs/arrow-fx-coroutines/arrow.fx.coroutines/on-cancel.html)

Register an `onCancel` handler after `fa`

➡️ only be invoked when the coroutine is cancelled

```kotlin=
suspend fun <A> onCancel(
  fa: suspend () -> A, 
  onCancel: suspend () -> Unit
): A // pass `fa` to `guaranteeCase` and only invoke `onCancel` when `ExitCase.Cancelled`
```

--

```kotlin=
import arrow.fx.coroutines.never
import arrow.fx.coroutines.onCancel
import kotlinx.coroutines.async
import kotlinx.coroutines.coroutineScope

suspend fun main() {
    coroutineScope {
        val job = async {
            onCancel(
                fa = { never<Unit>() },
                onCancel = { println("Cancelled!") } // will run
            )
        }

        job.cancel()
    }
}
```

---

### [CircuitBreaker](https://arrow-kt.io/docs/apidocs/arrow-fx-coroutines/arrow.fx.coroutines/-circuit-breaker/index.html)

<font size="6">🔍 Detect failures and prevent a failure from constantly recurring</font>

➡️ **Protect** resources or services from being overloaded

--

`CircuitBreaker` has 3 `CircuitBreaker.State`
- `Closed`
  - ➡️ normal state
- `Open`
  - ➡️ short-circuit/fail-fast all requests
- `HalfOpen`
  - ➡️ while it's allowing a request to go through, as a *test request*
  
--

[Source and Sample Code](https://github.com/arrow-kt/arrow/blob/main/arrow-libs/fx/arrow-fx-coroutines/src/commonMain/kotlin/arrow/fx/coroutines/CircuitBreaker.kt)

```kotlin=
import arrow.core.Either
import arrow.fx.coroutines.CircuitBreaker
import kotlinx.coroutines.delay
import kotlin.time.Duration
import kotlin.time.ExperimentalTime

@ExperimentalTime
suspend fun main() {
    val circuitBreaker = CircuitBreaker.of(
        maxFailures = 2,
        resetTimeout = Duration.seconds(2),
        exponentialBackoffFactor = 1.2,
        maxResetTimeout = Duration.seconds(60),
    )
    circuitBreaker.protectOrThrow { "I am in Closed: ${circuitBreaker.state()}" }.also(::println)

    println("Service getting overloaded...")

    // When an exception occurs it increments the failure counter
    // A successful request will reset the failure counter to zero
    Either.catch { circuitBreaker.protectOrThrow { throw RuntimeException("Service overloaded") } }.also(::println)
    Either.catch { circuitBreaker.protectOrThrow { throw RuntimeException("Service overloaded") } }.also(::println)
    println("Reach the maxFailures threshold = 2")
    circuitBreaker.protectEither { }.also { println("I am Open and short-circuit with ${it}. ${circuitBreaker.state()}") }

    println("Service recovering...").also { delay(2000) }

    println("After 2 seconds resetTimeout passed, allowing one request to go through as a test")
    circuitBreaker.protectOrThrow { "I am running test-request in HalfOpen: ${circuitBreaker.state()}" }.also(::println)
    println("I am back to normal state closed ${circuitBreaker.state()}")

    // I am in Closed: Closed(failures=0)
    // Service getting overloaded...
    // Either.Left(java.lang.RuntimeException: Service overloaded)
    // Either.Left(java.lang.RuntimeException: Service overloaded)
    // Reach the maxFailures threshold = 2
    // I am Open and short-circuit with Either.Left(arrow.fx.coroutines.CircuitBreaker$ExecutionRejected). CircuitBreaker.State.Open(startedAt=1635710570531, resetTimeoutNanos=2.0E9, expiresAt=1635710572531)
    // Service recovering...
    // After 2 seconds resetTimeout passed, allowing one request to go through as a test
    // I am running test-request in HalfOpen: HalfOpen(resetTimeoutNanos=2.0E9)
    // I am back to normal state closed Closed(failures=0)
}
```  

---

### Recap #1

- Safe Resource Handling leverages Immutability
  - `Atomic`, `Mutex`, `Semaphore`, `Channel`, ...
- Handle Resource
  - `bracket` & `bracketCase` ➡️ `acquire`, `use`, `release`
  - `Resource` ➡️ `acquire`, `release`
    - guarantee release in the correct **order** when `Cancelled` or `Failure`

--

### Recap #2

- Handle Effect
  - `guarantee` & `guaranteeCase` ➡️ `fa`, `finalizer`
  - `onCancel` ➡️ `fa`, `onCancel`
- `CircuitBreaker` ➡️ `Closed`, `Open`, `HalfOpen`
  - **Protect** resources or services from being overloaded
