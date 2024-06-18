# 17장 셀렉트 select

가장 먼저 완료되는 코루틴의 결과를 기다리는 함수

* 사용 예
  * 여러 개의 채널 중 버퍼에 남은 공간이 있는 채널을 확인하여 데이터 보내기
  * 이용 가능한 원소가 있는 채널로부터 데이터를 받을 수 있는지 여부 확인하기
  * 코루틴 사이에 경합 일으키기
  * 여러 개의 데이터 소스로 부터 나오는 결과값 합치기

실제로 사용하는 경우는 드물다,,

```kotlin
public suspend inline fun <R> select(crossinline builder: SelectBuilder<R>.() -> Unit): R {
    contract {
        callsInPlace(builder, InvocationKind.EXACTLY_ONCE)
    }
    return SelectImplementation<R>(coroutineContext).run {
        builder(this)
        doSelect()
    }
}
```



## 지연되는 값 선택하기

여러 개의 소스에 데이터를 요청한 뒤 가장 빠른 응답만 얻는 경우

여러 개의 비동기 프로세스로 시작한 뒤, select 함수를 표현식으로 사용, 표현식 내부에서 값 기다리기

select 내부에서는 셀렉트 표현식에서나올 수 있는 값을 명시하는 Deferred 값의 onAwait 함수를 호출

```kotlin
suspend fun requestData1() {
    delay(100_000)
    return "Data1"
}

suspend fun requestData2() {
    delay(1000)
    return "Data2"
}

val scope = CoroutineScope(SupervisorJob())

suspend fun askMultipleFroData() {
    val defData1 = scope.async { requestData1() }
    val defData2 = scope.async { requestData2() }
    
    return select {
        defData1.onAwait { it }
        defData2.onAwait { it }
    }
}

suspend fun main() = coroutineScope {
    println(askMultipleForData())
}

// (1초 후)
// Data2
```





####

```kotlin
suspend fun askMultipleForData() = coroutineScope {
    select<String> {
        async { requestData1() }.onAwait { it }
        async { requestData2() }.onAwait { it }
    }
}

suspend fun main() = coroutineScope {
    println(askMultipleForData())
}

// (100초 후)
// Data2
```



#### 코루틴끼리 경합 - async와 select

스코프를 명시적으로 취소 해야 함

```kotlin
suspend fun askMultipleForData() = coroutineScope {
    select<String> {
        async { requestData1() }.onAwait { it }
        async { requestData2() }.onAwait { it }
    }.also { coroutineContext.cancelChildren() }
}

suspend fun main() = coroutineScope {
    println(askMultipleForData())
}

// (1초 후)
// Data2
```



```kotlin
suspend fun askMultipleForData() = raceOf({
    requestData1()
}, {
    requestData2()
})

suspend fun main() = coroutineScope {
    println(askMultipleForData())
}

// (1초 후)
// Data2
```



## 채널에서 값 선택하기

* onReceive&#x20;
  * 채널이 값을 가지고 있을 때 선택됨
  * 값을 받은 뒤 람다식의 인자로 사용
  * select는 람다식의 결과값을 반환
* onReceiveCatching
  * 채널이 값을 가지고 있거나 닫혔을 때 선택됨
  * 값을 나타내거나 채널이 닫혔다는 걸 알려주는 ChannelResult를 받고, 해당 값을 람다식의 인자로 사용
  * select는 람다식의 결과값을 반환
* onSend
  * 채널의 버퍼에 공간이 있을 때 선택됨
  * 채널에 값을 보낸 뒤, 채널의 참조값으로 람다식 수행
  * select는 Unit을 반환

#### onReceive, onReceiveCatching 사용 예

```kotlin
suspend fun CoroutineScope.produceString(s: String,time: Long) = 
    produce {
        while(true) {
            delay(time)
            send(s)
        }
    }

fun main() = runBlocking {
    val fooChannel = produceString("foo", 210L)
    val barChannel = produceString("BAR", 500L)
    
    repeat(7) {
        select {
            fooChannel.onReceive {
                println("From fooChannel: $it")
            }
            barChannel.onReceive {
                println("From barChannel: $it")
            }
        }
    }
    
    coroutineContext.cancelChildren()
}
```

#### onSend 사용 예

#### 셀렉트 함수에서 호출, 버퍼에 공간이 있는 채널을 선택해 데이터를 전송하는 용도로 사용 가능

```kotlin
fun main() = runBlocking {
    val c1 = Channel<Char>(capacity = 2)
    val c2 = Channel<Char>(capacity = 2)

    // 값을 보냄    
    launch {
        for (c in 'A'..'H') {
            delay(400)
            select<Unit> { // select 함수 내 onSend 호출
                c1.onSend(c) { println("Sent $c to 1") }
                c2.onSend(c) { println("Sent $c to 2") }
            }
        }
    }
    
    // 값을 받음
    launch {
        while(true) {
            delay(1000)
            val c = select<String> {
                c1.onReceive { "$it from 1" }
                c2.onReceive { "$it from 2" }
            }
            println("Received $c")
        }
    }
}
```

