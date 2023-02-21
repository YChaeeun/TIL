# Kotlin Coroutine의 이해와 활용

{% embed url="https://github.com/kildongs/NaverCodeLAB" %}

{% embed url="https://kotlinlang.org/docs/coroutines-guide.html" %}



Concurrency 동시

* 하나의 스레드에서 시간을 나눠 쓰는 것
* 논리적인 개념

Parallelism 병렬성

* 여러 개의 코어, 각각 동작
* 물리적인 개념

Android 의 비동기 Task 처리

* AsyncTask / AsyncLoader / CountDownTimer
* HandlerThread / IntentService
* JobScheduler / WorkManager / DownloadManager
* Coroutine

### Kotlin Coroutine --> light-weight threads

* task run in a **scope**
  * activity 나 fragment 가 lifecycle scope 를 갖는 것처럼, thread 도 scope 를 가지게 되었다고 이해하면 됨!
* task run by threadPool
  * **task 를 정의한 쪽이랑 실행하는 쪽을 분리**
* task return a Job or Deferred

## Java Thread

*   무한으로 만들 수 없음&#x20;

    * 시스템 메모리(커널 메모리) 영역을 잡아먹음
    * 많을 수록 context switching 비용 증가


* Thread Pool
  * Thread 의 개수를 어느정도 범위에서 유지하고, 생성된 thread 를 재활용
  * 개수를 정해두고, 작업큐에 들어오는 작업을 스레드가 처리, 다 끝난 스레드는 새로운 작업을 작업 큐에서 가져와서 처리 --> 작업 요청이 급격하게 늘어나도 일정 부하 유지
  * maxPoolSize / corePoolSize
  * ThreadPoolExecuter()
    * &#x20;정교한 작업을 하려면 이 부분을 직접 만들어줘야해
  * Runnable / Callable / Future
    * Future : 비동기 task 가 생성됐을 때 반환되는 Handle / task 완료되면 결과에 접근 가능
  *   CompletableFuture

      * 미래에 처리할 task 로서, task 결과값을 리턴하거나 다른 task 를 이어서 trigger 시킴
      * 넘 메서드도 많고 복잡함 @\_@


* 동기화
  * Lock (ReenterLock)
  * Atomic Variable
    * ReadWriteLock
    * AtomicBoolean / AtomicInteger / AtomicLong / AtomicReference
  *   CountDownLatch

      * 완료될 task 의 수를 정하고, 완료될 때마다 감소, Await() 다 처리될 때까지 기다림


* Channel&#x20;
  * Thread 간 데이터 전송
  * PipedStream(오디오) / BlockingQueue(센서) / NIO Pipe.Channel (비디오영상처리)
    * 전달하려는 데이터 종류와 특성에 따라 더 최적화된 방식이 있음
  * 동일 thread 내에 send() / receive() 호출하면 Deadlock 발생 -- Thread 사이에서 써야 해
  *   Blocking Queue

      * one thread produce object --> another thread consumes
      * produce : Queue limit 까지 put 하고, 다 차면 block
      * consume : Queue 가 빌때까지 take, 비면 block


* ForkJoinPool
  * JAVA 7
  * task 를 쪼개서 실행하고 (fork)
  * 각각 실행한 결과를 취합 (join)

## Coroutine

* Task 정의
  * launch 로 실행할지, async 로 실행할지 task 를 정의 -- Coroutine Scope 에 쌓여(?) 있음
* Task 실행
  * Dispatcher 가 task 를 실행 -- thread pool (Executor Service)로 이루어져 있음!

### launch

* task 를 수행하며 결과를 받지 않음 (fire-forget 방식)
* Job 를 리턴 -- 현재 상태를 보고 cancel 할 수 있음
* Exception 이 발생하면 CoroutineScope 로 전달 - 내부에서 try-catch 로 하면 안잡힌대
* Job
  * launch 의 리턴 값
  * task 의 시작, 취소, 종료 대기 등을 할 수 있음
  * isActive, isCompleted 를 통해 task 상태 참조 가능

async

* task 를 수행하며 결과를 받음
* Defered\<T>를 반환, await 를 통해서 수행이 완료될 때 까지 기다림
* Exception 이 발생하면 거기서 중지 빠져나감, await 에서 결과 try-catch 잡아줘야 함
* Defered
  * async 의 결과값
  * await() -- task 가 끝나고 결과를 전달할 때 까지 대기

suspend

* 일반 함수가 아니고, 코루틴 내에서 써야해, 를 표시하는 거

runBlocking

* 새로운 CoroutineScope 를 만들어서 그 안에 있는 코드를 실행시켜주는 빌더
* 해당 블록에 있는 코루틴들이 모두 실행되고, 종료될 때까지 반환하지 않음

deley

* 해당 시간만큼 지연 후 실행
* Thread.sleep 이랑 비슷한데, 해당 thread 를 그냥 block 시키는 게 아니라 제어권을 넘기고 쉬는 것

### **Dispatcher**

* Task 가 어느 Thread 에서 실행될지를 결정하고 실행
* Default
  * 일반 연산, cpu 사용이 많은 작업에 적합
  * launch(Dispatcher.Default) {...}  == GlobalScope.launch {...}
* Main(UI)
* IO
  * 네트워크 작업, 파일 작업과 같은 I/O 작업
* Unconfined
  * 코루틴이 suspend 되었다가 상태가 재시작되면 적절한 thread 에 재할당



### Scope

* MainScope
  * UI 컴포넌트를 위한 scope
  * SupervisorJob + Dispatchers.Main
* viewModelScope
  * Job + Dispatchers.IO
  * ViewModel 내에 이미 Job이랑 Scope 를 정의, 알아서 clear 까지 해줌
* CoroutineScope
  * 프로세스의 생명 주기와 동일한 GlobalScope



withContext

* 새로운 CoroutineContext 를 만들어서 새로운 코루틴을 실행함&#x20;
* async {}.await() 같은 역할
* scope 내에서 UI 반영 등에 사용



