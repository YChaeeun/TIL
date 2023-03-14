# Kotlin Weekly #600 - My attempt on explaining Kotlin Coroutines

https://stefma.medium.com/my-attempt-on-explaining-kotlin-coroutines-eb2897d2f399

## 코루틴을 이해하기

코루틴을 이해하기 가장 쉬운 예시!

```kotlin
class UpdateDiskAfterSomeDependencyCollectUpdater(
	private val someDependency: SomeDependency,
	private val diskUpdater: DiskUpdater
) {
	suspend fun update() {
		someDependency.aflow
			.onEach { diskUpdater.writeToDisk(it) }
			.collect()
	}

	interface SomeDependency {
		val aflow: Flow<String>
	}

	interface DiskUpdater {
		fun writeToDisk(textToWrite: String)
	}
}
```

### update()가 하는 일

* `SomeDependency` 에서 `Flow`를 가져옴
* `Flow` 가 `emit` 으로 값을 방출하면, 해당 방출된 String 값마다 `diskUpdater.writeToDisk(it)` 를 호출
* `collect()` 로 값을 수집

## 테스트 작성

```kotlin
var someDependencyUpdate = MutableSharedFlow<String>()

val fakeSomeDependency = object : SomeDependency {
	override val aflow: Flow<String> = someDependencyUpdate
}

val fakeDiskUpdater = object : DiskUpdater {
	var writeToDiskCalled = false
	override fun writeToDisk(textToWrite: String) {
		writeToDiskCalled = true
	}
}

@Test  
fun `after emit of some dependency diskUpdater should write to disk`() = runTest {  
	val updater = UpdateDiskAfterSomeDependencyCollectUpdater(  
		someDependency = fakeSomeDependency,  
		diskUpdater = fakeDiskUpdater  
	)  
  
	updater.update()  
  
	someDependencyUpdate.emit("Saves this please")  
  
	assert(fakeDiskUpdater.writeToDiskCalled)  
}
```

### 위 테스트의 문제점

* 테스트가 막힘
* 가짜 객체를 만들고 > `runTest{}` 로 테스트를 시작하고 > `suspend` 함수인 `update()` 를 호출
  * 이때, `suspend` 함수가 문제

#### `suspend` 함수

* 중단함수
* 해당 함수가 이후에 일시정지하거나 재시작 할 수 있음
* 내 일이 끝날 때까지, 다음 코드를 실행하지 않는다
  * 내 일은 스레드를 막지 않아서 **다른 일**을 할 수 있는 상태
  * 내 일이 끝나면 스레드에게 일시정지한 시점에서부터 재시작해서 다음 코드를 실행하라고 알려줌

#### 위 테스트 동작을 다시 이해해보자

`suspend` 함수인 `update()`를 호출 > `update()`는 내부에서 `suspend` 함수인 `collect()` 를 호출

* 그렇게 되면 `collect()`는 "내 일이 끝날 때까지 다음 코드를 실행하지 않는다" 상태가 된다
  * 내 일이 끝난다는 건,
    * `Flow`가 `done` 완료/종료되거나(값을 더이상 방출하지 않음)
    * 코루틴이 `canceled` 취소되거나
    * `scope()` 가 `canceled` 취소되거나
* 테스트 코드에서는 위 세가지 상황 중 어떤 것도 만족하지 않아서, 코드가 실행되지 않는다
  * `Flow` 인 `someDependencyUpdate` 는 완료/종료되지 않고
  * 코루틴도 취소되지 않고
  * `scope` 인 `runTest()`함수로 생성된 `TestScope` 도 취소되지 않는다

## 문제를 해결하자!

* `scope` 취소하기
  * 말도 안됨
  * 테스트를 하려면 `TestScope` 내에서 이뤄져야 함
* `flow` `finish` 끝내기
  * 뒤에서 다룸
* 코루틴 취소하기
  * 취소하기 위해서는 `job.cancel` 이나 `scope.cancel`
    * `scope.cancel`은 하위 자식 코루틴까지 모두 취소
    * 하지만 `runTest`에서 제공하는 `TestScope`를 취소하거나, `TestScope`의 일부인 `job`을 취소 하게 되면 전체 테스트가 취소됨
  * 그렇다면 새로운 코루틴을 생성해서 이걸 `suspend` 하고 취소하면 어떨까?

### 첫번째 시도

새로운 코루틴을 생성하기 기존에 이미 있는 코루틴 빌더 `launch` 를 사용

```kotlin
val updater = UpdateDiskAfterSomeDependencyCollectUpdater(  
	someDependency = fakeSomeDependency,  
	diskUpdater = fakeDiskUpdater  
)  
  
launch {  
	updater.update()  
}  
  
someDependencyUpdate.emit("Saves this please")  
  
assert(fakeDiskUpdater.writeToDiskCalled)
```

#### 실패

* 새로운 코루틴(`launch` 내부 코드)이 너무 늦게 실행됨
  * 수행하기도 전에 "Assertion Failed" 에러로 테스트 끝;;
* `runTest{}` 블록 내부 코드들도 `suspend` 함수인데, 스레드가 "suspending point"에 진입하지 않음
  * 그래서 해당 스레드가 "다른 일"을 할 수 없음 (updater.update()를 수행할 수 없음)
  * 다른 코루틴을 수행하기 전에 "Assertion Failed" 에러로 코루틴이 취소되어버림
* 대신 `job.join()` 혹은 `testScope.advanceUntilIdle()` 를 사용해서 새로운 코루틴이 수행되도록 `suspend` 시킬 수 있음
  * `job.join()` : `job`이 완료될 때까지 코루틴을 `suspend` 시킨다
    * 하지만 새로 생성한 코루틴은 never-ending 이기때문에 이걸 사용하면 테스트가 다시 막히게 될 것,,
  * `testScope.advanceUntilIdle()` 도 마찬가지
* 코루틴이 실행될 때까지 기다려야지 완료될 때 까지 기다릴 수는 없음!ㅠ
  * 그렇다면 테스트 코루틴을 잠시 기다리게 해서 다른 코루틴을 수행할 수 있도록 해주는 `suspend` 함수를 추가하고, 이후에 다시 테스트 코루틴이 수행되는 시점으로 "jump" 되돌아오게 하는 건 어떨까?

### 두번째 시도

`suspending` 함수인 `delay`를 사용해서 테스트 코루틴을 조금 기다리게 하기

```kotlin
val updater = UpdateDiskAfterSomeDependencyCollectUpdater(  
someDependency = fakeSomeDependency,  
diskUpdater = fakeDiskUpdater  
)  
  
launch {  
	updater.update()  
}  
  
delay(1)  
someDependencyUpdate.emit("Saves this please")  
  
assert(fakeDiskUpdater.writeToDiskCalled)
```

#### 실패

launch 내부 코드를 실행할 수는 있지만 여전히 테스트가 중단된다,,!!

* 코드는 위에서 아래로 실행되는데, 코루틴 빌더인 `launch`를 만나게 되면 동작이 살짝 달라진다
  * 스레드는 해당 코드를 내부의 "task list" 에 넣는다
    * 스레드는 task 들의 목록이 있고, 앞에 task가 끝나면 그 다음 task를 수행하는 식으로 동작
  * 그러다가 `suspend` 함수인 `delay` (suspending point!) 를 만나게 된다
  * 이제 스레드는 "다른 일"을 할 수 있고, `suspend` 함수가 신호를 주면 재시작한다
    * 일시정지하지 않고, task list 에 있는 task 를 수행한다 - `updater.update()` 호출
    * `update()` 내부에서 `suspend` 함수를 다시 호출하므로 또 다른 suspending point가 생기고 스레드는 또 다시 다른 일을 할 수 있게 된다
  * `delay` 가 `suspending`을 완료했다고 스레드에 알려주고, 스레드는 다시 코드를 수행한다 - `someDependencyUpdate.emit` 과 `assert` 호출하기

왜 중단될까?

* 새로 생성된 코루틴은 테스트 코루틴을 수행하는 `TestScope`의 자식이고, 해당 `scope` 나 테스트 코루틴은 모든 자식이 취소/완료 될 때까지 취소되지 않기 때문이다
* 또한 `Flow` 는 절대 finish 완료되지 않는다

그렇다면 모든 자식 코루틴을 취소하면 되지 않을까?

### 마지막 시도 (성공)

필요없어지는 순간 never-ending `suspending` 코루틴을 취소시켜버리자!

```kotlin
val updater = UpdateDiskAfterSomeDependencyCollectUpdater(  
	someDependency = fakeSomeDependency,  
	diskUpdater = fakeDiskUpdater  
)  
  
val updateJob = launch {  
	updater.update()  
}  
  
delay(1)  
someDependencyUpdate.emit("Saves this please")  
updateJob.cancel()  
  
assert(fakeDiskUpdater.writeToDiskCalled)
```

명시적으로 코루틴을 취소하기 때문에 `TestScope` 나 해당 스코프의 테스트 코루틴이 예상한대로 취소되고 테스트가 중단되지 않는다

### 왜 이게 좋은 예시일까?

`suspend` 함수의 동작을 잘 보여주고 있기 때문

* 동작을 지연시키거나 일시정지
* 스레드를 막지 않으므로 스레드가 다른 일을 할 수 있다 ex) 다른 `suspend` 함수를 실행시키는 코루틴 실행하기

### 다른 해결 방법 세가지 (권장하지 않음🤔)

### "Flow" 를 종료하기

```kotlin
val someDependencyUpdate = MutableSharedFlow<String>(replay = 1)  

val fakeSomeDependency = object : SomeDependency {  
	override val aflow: Flow<String> = someDependencyUpdate.take(1)  
}  

val updater = UpdateDiskAfterSomeDependencyCollectUpdater(  
	someDependency = fakeSomeDependency,  
	diskUpdater = fakeDiskUpdater  
)  

someDependencyUpdate.emit("Saves this please")  
  
updater.update()  
  
assert(fakeDiskUpdater.writeToDiskCalled)
```

* `MutablesharedFlow`에 `replay = 1` 추가하기
  * 어떤 새로운 collector도 가장 최신의 값을 받음
  * 혹은 `MutableStateFlow` 사용하기
* `SomeDependency` 내부의 `Flow` 에게 첫번째 아이템을 받자마자 종료/취소하라고 알려주기
