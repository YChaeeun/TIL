# 9장 취소 cancellation

## 기본적인 취소 - cancel()

```kotlin
public fun cancel(cause: CancellationException? = null)
```

* 호출한 코루틴은 첫 번째 중단점에서 job을 끝낸다
* Job 이 자식을 가지고 있다면, 자식도 취소 (단, 부모는 영향을 받지 않음)
* 취소된 Job은 새로운 코루틴의 부모로 사용될 수 없음 (Cancelling > Cancelled 상태로 바뀜)

```kotlin
suspend fun main() = coroutineScope {
    val job = launch {
        repeat(1_000) { i ->
            delay(200)
            println("Printing $i")
        }
    }
    
    delay(1100) // job 작업을 기다려주기 위해 필요 ?
    job.cancel()
    job.join() // 왜 필요할까??
    println("Cancelled successfully")
}

// Printing 0
// Printing 1
// Printing 2
// Printing 3
// Printing 4
// Cancelled Successfullly
```

### 취소 후 join() 이 필요한 이유

* join()을 추가하면 취소를 마칠 때까지 중단되므로, 경쟁 상태가 발생하지 않는다
  * 경쟁 상태(race condition) - 공유 자원에 대해 여러 프로세스가 동시에 접근을 시도할 때, 실행 순서에 따라 결과값이 달라질 수 있는 상태

```kotlin
suspend fun main() = coroutineScope {
    val job = launch {
        repeat(1_000) { i ->
            delay(100)
            Thread.sleep(100) // 시간이 오래 걸리는 연산이라고 가정
            println("Printing $i")
        }
    }
    
    delay(1100)
    job.cancel()
    // job.join() 
    println("Cancelled successfully")
}

// Printing 0
// Printing 1
// Printing 2
// Printing 3
// Cancelled Successfullly 
// Printing 4 // join이 없다면 경쟁 상태가 발생 가능;
```

### Job.cancelAndJoin()

* kotinx.coroutine 라이브러리에서 cancel과 join 을 합친 확장함수도 제공

```kotlin
public suspend fun Job.cancelAndJoin() {
    cancel()
    return join()
}
```



### 한꺼번에 취소하기

* Job() 팩토리 함수로 생성된 job은 아래와 같은 방법으로 취소할 수 있다
* job에 딸린 수많은 코루틴을 한 번에 취소할 때 자주 사용!

```kotlin
suspend fun main() = coroutineScope {
    val job = Job()
    launch(job) {
        repeat(1_000) { i -> 
            delay(200)
            println("Printing $i")
        }
    }
    
    delay(1100)
    job.cancelAndJoin()
    println("Cancelled successfully")
}

// Printing 0
// Printing 1
// Printing 2
// Printing 3
// Printing 4
// Cancelled Successfullly
```



## 취소는 어떻게 작동할까?

* Job 이 취소되면 > 'Cancelling' 상태로 바뀜 > 상태가 바뀐 뒤 첫번째 중단점에서 CancellationExeption 예외를 던짐&#x20;
  * 예외는 try-catch 구문으로 잡을 수도 있지만, 다시 던지는 것이 좋다 (?)

```kotlin
suspend fun main() = coroutineScope {
    val job = Job()
    launch(job) {
        try {
            repeat(1_000) { i -> 
                delay(200)
                println("Printing $i")
            }
        } catch(e: CancellationException) {
            println(e)
            throw e // 다시 던지기?
        }
    }
    
    delay(1100)
    job.cancelAndJoin()
    println("Cancelled successfully")
    delay(1000)
}

// Printing 0
// Printing 1
// Printing 2
// Printing 3
// Printing 4
// JaobCancellationException..
// Cancelled Successfullly
```

* 취소된 코루틴은 단지 멈추는 것이 아니라 **내부적으로 예외를 사용**해 취소된다는 것을 명심!
  * 따라서 finally 블록 안에서 정리 할 수도 있음 ex) 파일이나 데이터베이스 연결 닫기

```kotlin
suspend fun main() = coroutineScope {
    val job = Job()
    launch(job) {
        try {
            delay(Randoem.nextLong(2000))
            println("Done")
        } finally {
            print("Will always be printed")
        }
    }
    delay(1000)
    job.cancelAndJoin()
}

// Will always be printed
// 또는,,
// (Done)
// Will always be printed
```

## 취소 중 코루틴을 한 번 더 호출하기

* 코루틴은 모든 자원을 정리할 필요가 있는 한 계속해서 실행될 수 있지만, 정리 과정 중에는 중단을 허용하지 않음
  * 다른 코루틴을 시작하려고 하면 무시 / 중단하려고 하면 CancellationException 던짐

```kotlin
suspend fun main() = coroutineScope {
    val job = Job()
    launch(job) {
        try {
            delay(2000)
            println("Job is done")
        } finally {
            println("Finally")
            launch { // 무시됨
                println("Will not be printed")
            }
            delay(1000) // 예외 발생
            println("Will not be printed")
        }
    }
}

// (1초 후)
// Finally
// Cancel Done
```

* `withContext(NonCancellable)`
  * 이미 코루틴이 취소되었는데 꼭 중단함수를 호출해야하는 경우 (ex. 데이터베이스의 변경사항을 롤백)에 사용
  * 코드 블록의 컨텍스트를 변경한다
  * 취소할 수 없는 Job 인 NonCancellable을 사용하기때문에 블록 내부에서 job은 Active 한 상태, 중단함수 원하는 만큼 호출 가능

```kotlin
suspend fun main() = coroutineScope {
    val job = Job()
    launch(job) {
        try {
            delay(2000)
            println("Coroutine finished")
        } finally {
            println("Finally")
            withContext(NonCancellable) { // 취소할 수 없는 Job 
                delay(1000L)
                println("Cleanup done")
            }
        }
    }
    delay(100)
    job.cancelAndJoin()
    println("Done")
}

// (1초 후)
// Finally
// Cancel Done
```



## invokeOnCompletion

* job 이 'Completed' 나 'Cancelled'와 같은 마지막 상태에 도달했을 때 호출될 핸들러를 지정하는 메소드
  * 자원을 해제할 때 자주 사용됨
* 취소하는 중에 동기적으로 호출되며, 어떤 스레드에서 실행할지 결정할 수는 없다

```kotlin
suspend fun main() = coroutineScope {
    val job = launch {
        delay(1000)
    }
    job.invokeOnCompletion { exception: Throwable? -> 
        println("Finished")
    }
    delay(400)
    job.cancelAndJoin()
}

// Finished
```

* 예외의 종류
  * 잡이 예외없이 끝나면 null
  * 코루틴이 취소되었으면 CancellationException
  * 코루틴을 종료시킨 예외
* job이 invokeOnCompletion이 호출되기 전에 완료되었으면 핸들러는 즉시 호출된다

```kotlin
suspend fun main() = coroutineScope {
    val job = launch {
        delay(Random.nextLong(2400))
        println("Finished")
    }
    delay(800)
    
    job.invokeOnCompletion { exception: Throwable? -> 
        println("Will always be printed")
        println("The exception was: $exception")
    }
    
    delay(800)
    job.cancelAndJoin()
}

// Will always be printed
// The exception was: ~~~
// (또는)
// Finished
// Will always be printed
// The exception was: ~~~
```



## 중단될 수 없는 걸 중단하기

* 중단점이 없으면 취소를 할 수 없다
  * delay 대신 Thread.sleep 으로 중단점이 없는 예시 > 중단점이 없어서 취소가 안됨

```kotlin
suspend fun main() = coroutineScope {
    val job = Job()
    launch(job) {
        repeat(1_000) { i -> 
            Thrad.sleep(200) // 복잡한 작업을 수행중이라고 가정, 중단점 없음
            println("Printing $i")
        }
    }
    
    delay(1000)
    job.cancelAndJoin()
    println("Cancelled successfully")
    delay(1000)
}

// Printing 0
// Printing 1
// Printing 2
// ... (1000까지)
```



### 해결 방법 1) `yield()` 주기적으로 호출하기

* `yield()` - 코루틴을 중단하고 즉시 재실행하는 메소드
  * 중단점을 제공하는 셈이므로 취소 등등의 작업을 할 수 있는 기회가 생김
* 최상위 중단 함수로, 스코프가 필요하지 않음
* 단, 중단하고 재개하는 과정에서 스레드 풀을 가진 디스패쳐를 사용하면 스레드가 바뀌는 문제가 생길 수 있음 (12장 참고)
* CPU 사용량이 크거나 스레드를 블로킹하는 중단함수에서 자주 사용

```kotlin
suspend fun main() = coroutineScope {
    val job = Job()
    launch(job) {
        repeat(1_000) { i -> 
            Thrad.sleep(200) // 복잡한 작업을 수행중이라고 가정
            yeild()
            println("Printing $i")
        }
    }
    
    delay(1000)
    job.cancelAndJoin()
    println("Cancelled successfully")
    delay(1000)
}

// Printing 0
// Printing 1
// Printing 2
// Printing 3
// Printing 4
// Cancelled Successfullly
```

* 중단함수 내에도 CPU 집약적이거나 시간 집약적인 연산들이 있다면, 각 연산 사이에 yield 사용하면 좋다

```kotlin
suspend fun cpuIntensiveOperations() = 
    withContext(Dispatchers.Default) {
        cpuIntensiveOperation1()
        yield()
        cpuIntensiveOperation2()
        yield()
        cpuIntensiveOperation3()
    }
```



### 해결 방법 2) Job 상태 추적하고 연산 중단하기

* isActive() 메소드로 상태 확인하기
* 코루틴 라이브러리는 Active 상태를 확인하는 함수 제공
  * CoroutineScope는 coroutineContext 프로퍼티를 사용해 참조할 수 있는 컨텍스트를 가지고 있음

```kotlin
public val CoroutineScope.isActive: Boolean
    get() = coroutineContext[Job]?.isActive ?: true
```

```kotlin
suspend fun main() = coroutineScope {
    val job = Job()
    launch(job) {
        do {
            Thread.sleep(200)
            println("Printing")
        } while (isActive) // 상태를 보고 실행 중지
    }
    
    delay(1100)
    job.cancelAndJoin()
    println("Cancelled successfully")
}
```



### 해결 방법 3) ensureActive() 메소드로 상태확인 후 예외던지기

* Active 상태가 아니면 CancellationException 던져서 중단
* CoroutineScope 또는 CoroutineContext 또는 Job 에서 호출되어야함
* yield() 보다 가벼워서 더 선호되는 방식

```kotlin
public fun CoroutineContext.ensureActive() {
    get(Job)?.ensureActive()
}

public fun Job.ensureActive(): Unit {
    if (!isActive) throw getCancellationException()
}
```

```kotlin
suspend fun main() = coroutineScope {
    val job = Job()
    launch(job) {
        repeat(1_000) { i -> 
            Thrad.sleep(200) // 복잡한 작업을 수행중이라고 가정
            ensureActive() // Active 상태가 아니라면 예외 던짐
            println("Printing $i")
        }
    }
    
    delay(1000)
    job.cancelAndJoin()
    println("Cancelled successfully")
}
```



## suspendCancellableCoroutine

* CancellableContinuation\<T> 로 래핑
* invokeOnCancellation - 코루틴이 취소되었을 때 행동을 정의하는 데 사용

```kotlin
suspend fun someTask() = suspendCancellableCoroutine { cont ->
    cont.invokeOnCancellation {
        // 정리 작업 수행
    }
    // 나머지 구현 부분
}
```

```kotlin
public suspend inline fun <T> suspendCancellableCoroutine(
    crossinline block: (CancellableContinuation<T>) -> Unit
): T =
    suspendCoroutineUninterceptedOrReturn { uCont ->
        val cancellable = CancellableContinuationImpl(uCont.intercepted(), resumeMode = MODE_CANCELLABLE)
        /*
         * For non-atomic cancellation we setup parent-child relationship immediately
         * in case when `block` blocks the current thread (e.g. Rx2 with trampoline scheduler), but
         * properly supports cancellation.
         */
        cancellable.initCancellability()
        block(cancellable)
        cancellable.getResult()
    }
```





