# 쓰레드 Thread

## 프로세스와 쓰레드

* 프로세스 process
  * 실행 중인 프로그램
  * 프로그램을 실행하면 OS 로 부터 자원(메모리)를 할당받아 프로세스가 됨
* 쓰레드 thread
  * 프로세스의 자원을 이용해서 작업을 수행함
  * 각 프로세스에는 최소 하나 이상의 쓰레드가 존재하고, 둘 이상의 쓰레드를 가진 경우 multi-threaded process 가 됨
  * 생성 개수에 제한은 없지만 개별적 메모리 공간을 차지하므로, 프로세스의 메모리 한계에 따라 갯수가 제한 됨

### 멀티 태스킹 vs 멀티 쓰레딩

* 멀티 태스킹
  * 여러개의 프로세스가 동시에 실행 될 수 있음
*   멀티 쓰레딩

    * 하나의 프로세스 내에서 여러 쓰레드가 동시에 작업을 수행
    * CPU의 코어는 한 번에 단 하나의 작업만 수행할 수 있으므로, 실제 작업의 갯수는 코어의 개수와 일치&#x20;
      * 그런데 해야 하는 작업의 양(쓰레드 개수)는 많은데?
      * 그래서 각 코어는 짧은 시간 동안 여러 작업들을 번갈아가면서 수행 -> 여러 작업들이 모두 동시에 수행되는 것처럼 보이게 해줌



### 멀티 쓰레딩의 장점

* CPU의 사용률이 향상된다
  * 싱글 쓰레드(프로세스 하나 당 쓰레드 하나)의 경우 요청마다 새 프로세스를 새로 생성해야 함 -> 프로세스를 생성하는 것은 쓰레드를 생성하는 것보다 비용이 크다 (더 많은 시간과 메모리 공간이 필요)
* 자원을 보다 효율적으로 사용할 수 있다
* 사용자에 대한 응답성이 향상된다
  * 동시에 여러 작업이 수행되는 것처럼 보이니까 응답이 바로바로 가능
  * 싱글 쓰레드였으면 하나 끝날 때까지 다른 건 반응 X
* 작업이 분리되어 코드 간결해진다

### 멀티 쓰레딩의 단점

* 여러 쓰레드가 한 프로세스의 자원을 공유하기때문에, 동기화(synchronization), 교착상태(deadlock)과 같은 문제가 발생할 수 있다
  * 동기화 : 한 데이터에 접근하는 두 쓰레드가 있을 때 한쪽이 해당 데이터를 변경했는데, 다른 쪽은 그 변경사항을 받지 못해서 문제가 생길 수 있음 -- 이걸 막는다
  * 교착상태 : 이러지도 저러지도 못함, 두 쓰레드가 자원을 점유한 상태에서 서로 상대편이 점유한 자원을 사용하려고 무한 대기만 하는 상태

## 쓰레드 구현과 실행

1.  Thread 클래스 상속

    * 해당 클래스를 상속 받으면 다른 클래스를 상속받을 수가 없어서 주로 Runnable 인터페이스를 구현하는 방식을 많이 씀

    ```java
    class MyThread extends Thread {
        public void run() { }
    }

    MyThread t = new MyThread();
    ```
2.  Runnable 인터페이스 구현

    * 재사용성이 높고 코드의 일관성을 유지할 수 있어서 보다 객체지향적인 방법&#x20;

    ```java
    class MyThread implements Runnable {
        public void run() { }
    }

    Runnable r = new MyThread();
    Thread t = new Thread(r);

    // 혹은 한 줄로
    Thread t = new Thread(new MyThread());

    // 쓰레드 이름
        // Tread(Runnable target, String name)
        // Thread(String name)
        // void setNAme(String name)
    ```

### Thread의 실행 - start()

* Thread 를 생성했다고 해서 자동으로 실행되는 것이 아니고, start() 를 호출해야 만 실행된다
  * 호출되었다고 바로 실행되지 않고, 우선 실행 대기 상태에 있다가 자신의 차례가 되어야 실행된다
  * 실행대기 중인게 없으면 바로 실행
* 한 번 실행이 종료된 스레드는 다시 실행할 수 없다
  * 한 쓰레드 당 start() 는 한 번만 호출 가능

## start() 와 run()

### start()

* 새로운 쓰레드가 작업을 실행하는데 필요한 호출스택(call stack)을 새로 생성함
  * 그 다음에 run() 을 호출해서 새로 생성된 호출 스택에 run() 이 첫번째로 올라가게 해줌
* 모든 쓰레드는 독립적인 작업을 수행하기 위해 자신만의 호출스택이 필요하기 때문에 start() 로 호출스택을 생성
  * 종료되면 작업에 사용된 호출 스택은 소멸됨

### run()

* 만약 start() 를 호출하지 않고 run() 만 호출되었다면, 해당 쓰레드는 새로운 호출 스택이 없어서 호출된 쪽의 쓰레드의 호출스택에 남는다 (ex. main 쓰레드)

### main Thread

* main 메서드의 작업을 수행하는 쓰레드
* 간혹 main 메서드가 수행을 마쳐서 main 쓰레드가 종료되어도, 종료되지 않은 다른 쓰레드가 있어서 프로그램이 종료되지 않는 경우가 발생함
  * 프로그램은 실행 중인 사용자 쓰레드가 하나도 없을 때 종료된다

## 싱글 쓰레드와 멀티 쓰레드

### 싱글 쓰레드

* 한 작업을 마친 후 다음 작업을 수행한다
* 단순히 CPU만을 사용하는 계산 작업이라면 싱글 쓰레드가 더 효율이 좋다

### 멀티 쓰레드

* 짧은 시간동안 여러 쓰레드가 번갈아가면서 작업을 수행해서, 동시에 수행되는 것처럼 보이게 한다
* 두 쓰레드가 서로 다른 자원을 사용하는 작업의 경우, 멀티 쓰레드가 더 효율적이다
  * ex) 사용자에게 데이터를 입력받거나, 네트워크 파일 주고받기, 프린터로 파일 출력하기와 같은 I/O 작업
* 쓰레드를 번갈아가면서 바꾸는 context switching 도 시간을 잡아먹으므로 전체 수행 시간은 싱글 쓰레드보다 길 수도 있음
* **context switching**
  * 현재 진행 중인 작업의 상태를 저장하고 읽어옴
  * 다음에 실행해야 할 위치 (PC, program counter) 등등
* 병행 concurrent : 여러 쓰레드가 여러 작업을 동시에 진행
* 병렬 parallel : 하나의 작업을 여러 쓰레드가 나눠서 처리함

### OS 와 Thread

* 실행 중인 프로그램(프로세스)는 OS의 프로세스 스케줄러의 영향을 받기 때문에 매번 다른 실행 시간이 걸릴 수 있다
* JVM의 쓰레드 스케줄러에 의해서 어떤 쓰레드가 얼마동안 실행될 것인지 결정되는 것처럼, 프로세스도 프로세스 스케줄러에 의해서 실행 순서와 실행 시간이 결정됨
  * 따라서 상황에 따라서 프로세스와 쓰레드에 할당되는 실행 시간이 일정하지 않음
* 자바는 OS( 플랫폼) 독립적이라고들 하지만, 실제로는 쓰레드 처럼 몇몇 부분은 OS 종속적이다

## 쓰레드의 우선순위

*   우선순위 priority

    * 쓰레드가 수행하는 작업의 중요도와 우선순위에 따라 쓰레드는 서로 다른 우선순위를 가지고, 이 우선순위를 통해서 특정 쓰레드가 더 많은 작업 시간을 갖도록 할 수 있다

    ```java
    void setPriority(int newPriority)
    int getPriority()

    // 숫자가 높을 수록 높은 우선순위!
    public static final int MAX_PRIORITY = 10
    public static final int MIN_PRIORITY = 1
    public static final int NORM_PRIORITY = 5
    ```

## 쓰레드 그룹 thread group

* 관련된 쓰레드를 그룹으로 다루기 위함
  * 쓰레드 그룹 내에도 새로운 쓰레드 그룹을 포함할 수 있음
  * 보안 상 도입된 것으로, 자신이 속한 쓰레드 그룹이나 하위 쓰레드 그룹은 변경할 수 있지만, 다른 쓰레드 그룹의 쓰레드는 변경할 수 없다
* 모든 쓰레드는 반드시 쓰레드 그룹에 속해 있어야 함
  * 우리가 생성하는 모든 쓰레드 그룹은 main 쓰레드 그룹의 하위 쓰레드 그룹이 됨
  * 쓰레드 그룹을 지정하지 않고 생성하면 자동으로 main 쓰레드 그룹에 속하게 됨
* 자바 어플리케이션이 실행되면, JVM은 main과 system 이라는 쓰레드 그룹을 만들고, JVM 운영에 필요한 쓰레드들을 생성해서 이 쓰레드 그룹에 포함시킴
  * ex) main 이라는 쓰레드 그룹, system 이라는 쓰레드 그룹을 만들고,
    * main 쓰레드 그룹에는 메서드를 수행하는 main 이라는 이름의 쓰레드가 속하게 됨
    * system 쓰레드 그룹에는 가비지 컬렉션을 수행하는 Finalizer 쓰레드를 system 쓰레드 그룹에

```java
ThreadGroup getThreadGroup()     // 쓰레드 자신이 속한 쓰레드 그룹을 반환
void uncatuchException(Thread t, Throwable e) 
        // 쓰레드 그룹의 쓰레드가 처리되지 않은 예외로 실행이 종료되었을 때, JVM 에서 자동으로 호출됨

ThreadGroup tg1 = new ThreadGroup("Group1"); // Group1 이라는 이름의 쓰레드 그룹 생성
ThreadGroup sub1 = new ThreadGroupt(tg1, "SubGroup1"); 
                // Group1의 하위에 SubGroup1 이라는 이름의 쓰레드 그룹 생성
                
tg1.setMaxPriority(3); // 최대 우선순위를 3으로 변경
                        // 하위 쓰레드 그룹에도 영향을 미침

Runnable r = new Runnable() { public void run() { } }
new Thread(tg1, r, "th1");    // start thread
new Thread(sub1, r, "sub1");  // start thread

main.list(); // main 쓰레드 그룹에 속한 쓰레드와 하위 쓰레드에 대한 정보를 출력
```

## 데몬 쓰레드 (daemon thread)

* 일반 쓰레드의 작업을 돕는 보조 쓰레드
  * 일반 쓰레드가 모두 종료되면 같이 종료됨
  * ex) 가비지 컬렉터(Finalizer), 워드 프로세서의 자동 저장, 화면 자동 갱신 등등
    * getAllStackTraces() 를 사용해서 프로그램 처리에 필요해서 자동으로 생성된 데몬 쓰레드들을 확인 할 수 있다!
* 무한 루프와 조건문을 이용해서 실행 수 대기하고 있다가 특정 조건이 만족되면 작업을 수행하고 다시 대기
  * 무한 루프 돌고 있기 때문에 만약 이 쓰레드가 일반 쓰레드라면 강제 종료하지 않는 한 해당 프로그램은 영원히 종료되지 않는다 ;;
* 데몬 쓰레드가 생성한 쓰레드는 자동적으로 데몬 쓰레드가 된다

```java
boolean isDaemon()         // 해당 쓰레드가 데몬 쓰레드인지 확인, 맞으면 true 반환
void setDaemon(boolean on) // 쓰레드를 데몬쓰레드 혹은 일반 쓰레드로 변경 on=true 인 경우 데몬
                                // setDaemon 은 start() 를 호출하기 전에 실행되어야 한다

...
Thread t = new Thread(new ThreadEx());
t.setDaemon(true);
t.start();
....
```

## 쓰레드의 실행 제어

### 쓰레드 상태

* Thread의 getState()메서드를 호출해서 확인할 수 있다 (JDK1.5 이후부터)

| 상태                      | 설명                                                                                              |
| ----------------------- | ----------------------------------------------------------------------------------------------- |
| NEW                     | 쓰레드가 생성되고 아직 start()가 호출되지 않은 상태                                                                |
| RUNNABLE                | 실행 중 또는 실행 가능한 상태                                                                               |
| BLOCKED                 | 동기화블럭에 의해서 일시정지된 상태(lock 이 풀릴 때까지 대기)                                                           |
| WAITING, TIMED\_WAITING | <p>쓰레드의 작업이 종료되지는 않았지만, 실행 불가능한 일시정지 상태(unrunnable)</p><p>TIMED_WAITING은 일시정지시간이 지정된 경우를 의미</p> |
| TERMINATED              | 쓰레드의 작업이종료된 상태                                                                                  |

![쓰레드의 상태](../.gitbook/assets/Thread\_state.png)

> 번호를 붙이긴 했지만 이게 이 순서대로 실행된다는 뜻은 아님

1. start() 로 새로 생성된 NEW 상태인 쓰레드를 실행대기열(큐)에 넣어둔다
2. 실행대기열에 있는 RUNNABLE 쓰레드는 자신의 차례가 되면 실행상태가 된다
3. 주어진 실행 시간이 다되거나, yield() 를 만나면 다시 실행대기상태가 되고, 다음 쓰레드가 실행된다
4. 실행 중에 일시정지 상태가 될 수 있다
   * suspend(), sleep(), wait(), join(), I/O block 에 의해서 일시정지가 될 수 있음
   * I/O block : 입출력 작업 시 지연상태 ex) 사용자 입력 / 작업이 끝나면 다시 실행대기 상태가 됨
5. 지정된 일시정지 시간이 다 되거나(time-out), 메서드가 호출돼서 일시정지 상태를 벗어나면 다시 실행대기 상태가 된다
   * notify(), resume(), inturrupt()
6. 실행을 모두 마치거나, stop()이 호출되면 쓰레드는 소멸된다

### sleep(long millis) - 일정시간동안 쓰레드 멈춤

* 지정된 시간이 다 되거나, interrupt() 가 호출되면 잠에서 깨어나 **실행대기 상태**가 된다
  * interrupt() 호출은 즉 InterruptedException 를 발생시키는 거라서,  try-catch문 필수
* sleep()은 항상 현재 실행 중인 쓰레드에 대해 작동하기 때문에, `Thread.sleep(2000);` 과 같이 작성해줘야 함
  * `쓰레드 이름.sleep()` 처럼 참조 변수를 이용해 호출하면 안됨! 해당 이름의 쓰레드를 sleep 하는 게 아님 -0-

```java
static void sleep(long millis)
static void sleep(long millis, int nanos)

/* ------------------------------ */
void delay(long millis) {
    try {
        Thread.sleep(millis);
    } catch(InterruptedException e) { }
}
```

### interrupt() & interrupted() - 쓰레드의 작업을 취소

* interrupt()
  * 작업이 끝나기 전에 취소시켜야 할 때 사용
  * 단,, 멈추라고 요청을 하는 것 뿐이지 **강제로 종료시키지는 못함**
    * 그냥,, interruped상태(인스턴스 변수)를 바꾸는 것 뿐,,
  * 쓰레드가 sleep(), wait(), join()에 의해서 WAITING 상태에 있었을 때 해당 쓰레드에 대해 interrupt()를 호출하면, InterruptedException 이 발생하고 쓰레드는 RUNNABLE로 바뀜
* isInterrupted()
  * 단순 interrupt 상태 반환
*   interruped()

    * 해당 쓰레드에 대해 interrupt()가 호출되었는지 알려줌
    * 현재 쓰레드의 interrupted 상태를 **반환 후, false 로 변경**

    ```java
    Thread th = new Thread();
    th.start();
    ....
    th.interrupt();

    class MyThread extends Thread {
        public void run() {
            while(!interrupted()) { // interrupted=true 면 반복문 종료
            }
        }
    }


    // 만약 interrupt 상태 체크하는 거 안에서 interrupt()를 호출하는 메서드가 호출되면,
    // 해당 쓰레드의 상태 값이 다시 false 로 초기화 되기 때문에 바르게 종료가 되지 않음
    class MyThread extends Thread {
        public void run() {
            while(!isInterrupted()) {
                try {
                    sleep(1000); 
                    // 여기서 interrupt()가 호출되고 interrupt 상태가 false 로 초기화 된다
                    // 그러면 정상적으로 종료가 안됨
                } catch(InterruptedException e) {
                    interrupt(); // 그래서 이 부분을 추가해서 true 로 다시 값을 설정해줘야 함
                }
            }
        }
    }
    ```

### suspend(), resume(), stop()

* suspend() : 쓰레드를 멈춤, resume() 으로만 다시 실행대기 상태가 될 수 있음
* resume() : suspend()로 멈춰진 쓰레드를 다시 실행대기 상태로 만듬
* stop() : 호출되는 즉시 쓰레드 종료
* 쓰레드의 실행을 제어하는 데 가장 간단한 방법이었지만, deadlock을 발생시킬 위험이 크므로 현재는 deprecated 되었다
  * 사용하지 말 것!!
  * 남아있는 건 이전 코드와의 상호 호완성 때문,,

### yield() - 다른 쓰레드에게 양보

* 자신에게 주어진 실행 시간을 다음 차례의 쓰레드에게 양보한다
  * ex) 1초의 시간을 할당받은 쓰레드가 0.5 만큼 하다가 yield()가 호출되면, 나머지 0.5초 포기하고 실행 대기 상태가 됨
* yield(), interrupt() 를 적절하게 사용하면, 프로그램의 응답성을 높이고 효율적인 실행이 가능해진다
  * 아무일도 하지 않는데 반복문은 돌고 있는 상황(busy-waiting)일 때 yield()를 하면 시간을 낭비하지 않을 수 있음

### join() - 다른 쓰레드의 작업을 기다림

* 자신이 하던 작업을 잠시 멈추고, 다른 쓰레드가 지정된 시간동안 작업을 수행하도록 할 때 사용
  * 현재 쓰레드 작업 중에 다른 쓰레드의 작업이 먼저 수행되어야 할 때 사용한다!
  * 시간을 따로 지정하지 않으면 다른 쓰레드의 작업이 끝날 때까지 기다림
* interrupt() 에 의해서 대기 상태에서 벗어날 수 있음 (try-catch 문으로 감싸야 함)
  * 현재 쓰레드가 아닌 특정 쓰레드에 대해 동작하므로 static 메서드 아님

```java
void join()
void join(long millis)
void join(long millis, int nanos)

/* ----------------------------- */
try {
    th.join();  // 현재 실행 중인 쓰레드가 다른 쓰레드 th1의 작업이 끝날 때 까지 기다림
} catch (InterruptedException e) { }
```

* ex) JVM의 가비지 컬렉터가 동작할 때, 메인 쓰레드에서 가비지 컬렉터의 작업이 끝날 때 까지 join()으로 기다려서 메모리를 확보한 다음에 메인 쓰레드의 작업을 계속해야 한다

## 쓰레드의 동기화 synchronization

* 한 쓰레드가 진행 중인 작업을 다른 쓰레드가 간섭하지 못하도록 막는 것
  * 임계 영역(critical section) & 잠금, 락 (lock)
  * 공유 데이터를 사용하는 코드 영역을 임계 영역으로 지정해 놓고, 공유 데이터가 가지고 있는 lock를 획득한 단 하나의 쓰레드만 이 영역 내의 코드를 수정할 수 있게 한다
  * 그리고 임계 영역 내 작업을 마치고 lock을 반납해야만 다른 쓰레드가 반납된 lock을 획득하여 임계 영역의 코드를 수행

### synchronized를 이용한 동기화

* 방법 1. 메서드 전체를 임계 영역critical section으로 지정
  * 쓰레드는 synchronized 메서드가 호출된 시점부터 lock 을 얻어서 작업을 수행하다가 메서드가 종료되면 lock 을 반환한다
* 방법 2. 특정 영역을 임계critical section으로 지정
  * synchronized (참조 변수)는 락을 걸고자 하는 객체를 참조해야 하고, 이 블럭을 synchronized블럭이라고 부른다
  * 블럭의 영역에 들어가면 지정된 객체의 lock을 얻고, 벗어나면 lock을 반납
* 자동으로 lock을 얻고 반납하기 때문에 영역을 지정만 해주면 됨
  * critical section 은 멀티 쓰레드 프로그램의 성능을 좌우하기 때문에, 가능하면 critical section을 최소화해서 효율적인 프로그램을 작성해야 함

```java
// 메서드 전체를 임계 영역으로
public synchronized void calcSum() { }

// 특정한 영역을 임계 영역으로
synchronized(객체의 참조변수) { }
```



### wait() & notify()

* 특정 쓰레드가 락을 너무 오래 가지지 않도록 하는 것도 중요
  * 계속 잡고 있게 되면 다른 쓰레드는 작업을 못하고 계속 기다려야 함 ㅇㅅㅇ
* wait() & notify() 는 Object 클래스에 정의되어 있음
* wait()
  * 동기화된 임계 영역critical section의 코드를 수행하다가, 더 작업을 할 상황이 아니면 일단 wait()을 호출해서 lock을 반환하고 대기,,
  * 매개변수가 없을 경우 notify(), notifyAll() 이 호출될 때 까지 대기하지만, 매개변수 있을 경우 해당 시간만큼만 대기함 (자동으로 notify() 호출하는 거랑 같음)
* notify()
  * notify()를 호출하면 기다리던 쓰레드 중 하나가 lock을 다시 얻고 작업 진행
  * wait() 으로 반환 후에 notify()로 다시 얻어서 임계 영역에 진입하는 것을 재진입reenterance라고 함
* 단, 오래 기다린 쓰레드가 락을 얻는 다는 보장이 없음
  * wait()를 호출하면 해당 쓰레드는 lock을 반환하고 waiting pool에서 대기
  * notify()가 호출되면, 해당 객체의 waiting pool 에 있던 쓰레드 중에 임의의 쓰레드만 통지?를 받음
    * notifyAll() 로 waiting pool 에 있는 모든 쓰레드에게 통지를 할 수는 있는데 그래도 lock를 얻는 건 하나의 쓰레드

### 기아 starvation & 경쟁 race condition

* 기아 starvation
  * 쓰레드가 계속 lock을 받지 못하고 오래동안 기다리는 상태
  * notify() 대신 notifyAll() 를 호출하면 이 현상을 막을 수 있음 -- ?
* 경쟁 race condition
  * 여러 쓰레드가 lock을 얻기 위해서 경쟁하는 상태
  * 이를 개선하기 위해서는 쓰레드를 구분해서 통지하는 것이 필요함 --> Lock과 Condition을 이용한 동기화

## Lock과 Condition을 이용한 동기화

* 같은 메서드 내에서만 lock을 걸 수 있는 제약이 불편할 때 사용하는 lock 들

### ReentranctLock

* 재진입이 가능한 Lock&#x20;
  * wait() notify() 처럼 특정 조건에서 lock을 풀고 다시 lock을 얻을 수 있음
* 가장 배타적
  * lock 이 반드시 있어야만 임계 영역의 코드를 수행할 수 있음

### ReentrantReadWriteLock

* 읽기에는 공유적이고, 쓰기에는 배타적인 lock
  * 읽기 lock 이 이미 걸려 있어도, 중복적으로 lock을 걸 수 있음
    * 읽는 건 여러 쓰레드가 같이 읽어도 됨\~!
  * 단, 쓰기의 경우 값이 변경되는 거니까 쓰려면 쓰기 lock을 걸어야만 가능

### StampedLock

* ReentrantReadWriteLock에 낙관적인 lock 기능 추가
  * lock 을 걸고 해지할 때 스탬프(long 타입의 정수값)을 사용
  * 낙관적 읽기 lock (optimistic reading lock)
    * ReentrantReadWriteLock의 경우에 쓰기 lock을 얻으려면 읽기 lock이 풀릴 때 까지 기다려야 하는데, 낙관적 읽기 lock의 경우 쓰기 lock에 의해 바로 풀림
    * 따라서 낙관적 읽기에 실패하면, 읽기 lock을 얻어서 다시 읽어와야 함
  * 무조건 읽기 lock을 걸지 않고, **쓰기와 읽기가 충돌할 때만 쓰기가 끝난 후, 읽기 lock을 거는 것**

```java
int getBalance() {
    long stamp = lock.tryOptimisticRead(); // 낙관적 읽기 lock 걸기
    int curBalance = this.balance;    // 공유 데이터 읽기
    
    if (!lock.validate(stamp)) { // 쓰기 lock에 의해 낙관적 읽기 lock이 풀렸는지 확인
        stamp = lock.readLock();    // lock이 풀렸으면 읽기 lock 얻기위해서 대기
        
        try {
            curBalance = this.balance;  // 공유 데이터 다시 읽기
        } finally {
            lock.unlockRead(stame);    // 읽기 lock 해제
        }
    }
    
    return curBalance; // 낙관적 읽기 lock 이 풀리지 않았으면 읽어온 값 바로 반환
}
```



### ReentrantLock 생성자

```java
ReentrantLock()

ReentrantLock(boolean fair)
    // fair=true 일 때, 가장 오래 기다린 쓰레드가 lock을 획득할 수 있게 처리
    // 어떤 쓰레드가 가장 오래걸렸는지 확인이 필요하므로 성능이 떨어짐
```

```java
void lock()            // lock 잠금
void unlock()          // lock 해지
boolean isLocked()     // lock 이 잠겼는지 확인

// synchronized 블럭과 달리 수동으로 잠금과 해지가 이뤄지기 때문에 안까먹게 주의!!
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {

} finally {
    lock.unlock();
}
```



### ReentrantLock 과 Condition

*   Condtion을 만들어서 각각의 waiting pool 을 만들어서 대기하게 함

    * wait() & notify()로 쓰레드의 종류를 구분하지 않고, 공유 객체의 waiting pool 에 몰아 넣을 때 생길 수 있는 단점 해결 가능!
    * wait() 대신 await()&#x20;
      * await() / awaitUninterrupibly() / await(long, TimeUnit) / awaitNanos(long) / awaitUntil(Date)
    * notify(), notifyAll() 대신 signal() / signalAll()

    ```java
    private ReentrantLock lock = new ReentrnatLock();

    private Condition forCook = lock.newCondition();
    private Condition forCustomer = lock.newCondition();
    ```

## volatile

* 값을 캐시가 아닌 메모리에서 읽어오도록 함
  * 멀티 코어 프로세서에서는 코어별로 별도의 캐시를 가지고 있는데, 해당 캐시들은 메모리에 있는 값의 일부
  * 만약 메모리에 저장된 값이 갱신이 되었는데, 캐시가 갱신되지 않는다면 데이터 값 간의 불일치가 발생할 수 있음
    * volatile 을 붙이면 캐시가 아닌 메모리에서 값을 읽어오므로 불일치 해소 가능!
    * synchronized 블럭에서 들어갔다 나올 때도 메모리와 캐시 간 동기화가 이뤄지기 때문에 값의 불일치가 해소 됨

### volatile로 long과 double 원자화

* JVM은 데이터를 4 byte(=32bit) 단위로 처리하기 때문에, int나 int 보다 작은 타입들은 한번에 읽고쓰기 가능
  * 단 하나의 명령어로 읽거나 쓰기가 가능하다는 뜻
*   그러나 크기가 8 byte 인 long과 double타입 변수는 하나의 명령어로 값을 읽거나 쓸 수 없어서, 중간에 다른 쓰레드가 끼어들 여지가 있음

    * 이때 volatile을 붙이면 해당 변수에 대한 읽기와 쓰기가 **원자화** 됨 -- 작업을 더이상 나눌 수 없다는 뜻
    * 주의) 원자화와 동기화는 다름!&#x20;

    ```java
    volatile long balance;

    synchronized int getBalance() {
        return balance;
    }

    synchronized void withdraw(int money) {
        if(balance >= money) balance -= money
    }
    ```

## fork & join 프레임워크

