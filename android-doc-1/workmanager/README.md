# Workmanager

## Worker

* 백그라운드에서 수행할 동작 정의 **WHAT**
* Key-value 형태의 Input과 output 제공 (Data 로 주고받음)
* success, failure, retry 결과값 반환
  * Result.success() / Result.failure() / Result.retry()
  * retry시 정책 및 시간 설정 가능 - [setBackOffCriteria](https://developer.android.com/reference/kotlin/androidx/work/WorkRequest.Builder#setbackoffcriteria)
  *



## WorkRequest

* 어떻게, 언제 work를 수행할지 결정 **HOW, WHEN**
* OneTimeWorkRequest - 반복하지 않는 work&#x20;
  * Workmanager.enque / WorkManager.beginWith
* PeriodicWorkRequest - 반복하는 work
  * PeridodicWorkRequest.Builder - flexInterval 로 실행 시간 별 간격 조정 가능

```kotlin
val constraints = Constraints.Builder()
        .setRequiredNetworkType(NetworkType.CONNECTED)
        .build()

// Define the input
val imageData = workDataOf(Constants.KEY_IMAGE_URI to imageUriString)

// Bring it all together by creating the WorkRequest; this also sets the back off criteria
val uploadWorkRequest = OneTimeWorkRequestBuilder<UploadWorker>()
        .setInputData(imageData)
        .setConstraints(constraints)        
        .setBackoffCriteria(
                BackoffPolicy.LINEAR, 
                OneTimeWorkRequest.MIN_BACKOFF_MILLIS, 
                TimeUnit.MILLISECONDS)
        .build()
```



## WorkManager

* 백그라운드에서 작업처리
* work 수행 보장 (기기나 앱 종료 후 재시작할 경우에도)

#### 구성요소

* Internal TaskExecutor - enque work 실행
* WorkManager database - work와 관련된 모든 정보 저장 (ex.상태, input, output)
  * 기기 재시작과 같이 work 작업 중간에 방해를 받더라도 여기에서 데이터를 가져와서 작업을 수행
* WokerFactory - worker 객체 생성
* Default Executor - 기본 executor / 메인 스레드가 아닌 곳에서 동작

<figure><img src="../../.gitbook/assets/스크린샷 2024-07-15 오후 2.44.19.png" alt=""><figcaption></figcaption></figure>

`InternalTaskExecutor`가 `WorkRequest`정보를 `WorkManager Database`에 저장 > Constraint이 맞게되면(ex. 네트워크 연결), `InternalTaskExecutor`가 `WorkFactory`에게 `Worker` 객체 생성을 지시 > `Executor`가 `Worker`의 `doWork()` 실행



Worker 동작을 제어하고 싶다면 `ListenableWorker`를 커스텀해서 사용하면 된다?!

## Chain

작업 동시 수행 & 이어서 다음 작업 수행 등등

```kotlin
WorkManager.getInstance()
    .beginWith(Arrays.asList(
                             filterImageOneWorkRequest, 
                             filterImageTwoWorkRequest, 
                             filterImageThreeWorkRequest))
    .then(compressWorkRequest)
    .then(uploadWorkRequest)
    .enqueue()
```

동시 수행된 작업의 결과 합치기 - InputMerger, ArrayCreatingInputMerger

```kotlin
val compressWorkRequest = OneTimeWorkRequestBuilder<CompressWorker>()
        .setInputMerger(ArrayCreatingInputMerger::class.java)
        .setConstraints(constraints)
        .build()
```

<figure><img src="../../.gitbook/assets/스크린샷 2024-07-15 오후 2.57.25.png" alt=""><figcaption></figcaption></figure>

## WorkRequest 상태 관찰하기

[getWorkInfoByIdLiveData](https://developer.android.com/reference/androidx/work/WorkManager.html#getWorkInfoById\(java.util.UUID\))

```kotlin
WorkManager.getInstance().getWorkInfoByIdLiveData(uploadWorkRequest.id)
        .observe(lifecycleOwner, Observer { workInfo ->
            // Check if the current work's state is "successfully finished"
            if (workInfo != null && workInfo.state == WorkInfo.State.SUCCEEDED) {
                displayImage(workInfo.outputData.getString(KEY_IMAGE_URI))
            }
        })
```

* status
  * BLOCKED - 대기 중, chain에 다음 작업으로 등록되어있지는 않음
  * ENQUEUED - 대기 중, constraint 조건에 맞으면 동작할 준비 완료된 상태
  * RUNNING - 실행하고 있는 상태 doWork() 호출됨
    * Result.retry() 호출한 경우 다시 ENQUEUED 상태로 복귀
  * SUCCEEDED - doWork()가 Result.success() 반환함
  * FAILED - doWork()가 Result.failure() 반환함
  * CANCELLED

<figure><img src="../../.gitbook/assets/스크린샷 2024-07-15 오후 3.03.55.png" alt=""><figcaption></figcaption></figure>

