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
    
    delay(1100)
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

```kotlin
public suspend fun Job.cancelAndJoin() {
    cancel()
    return join()
}
```



### 한꺼번에 취소하기

```kotlin
suspend fun main() = coroutineScope {
    val job = Job()
    launch(job) {
        repeat(1_000) { i -> 
            delay(200)
            println("Printing $i")
        }
    }
    
}
```

