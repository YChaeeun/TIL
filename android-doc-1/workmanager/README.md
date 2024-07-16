# Workmanager

## Worker

* 백그라운드에서 수행할 동작 정의 **WHAT**
* Key-value 형태의 Input과 output 제공 ([Data ](https://developer.android.com/reference/androidx/work/Data?\_gl=1\*1j8x2lp\*\_up\*MQ..\*\_ga\*MzY2MzExODI2LjE3MjExMTg4NTU.\*\_ga\_6HH9YJMN9M\*MTcyMTExODg1NC4xLjAuMTcyMTExODg1NC4wLjAuMA..)로 주고받음)
* success, failure, retry 결과값 반환
  * Result.success() / Result.failure() / Result.retry()
  * retry시 delay 설정 가능 - [setBackOffCriteria](https://developer.android.com/reference/kotlin/androidx/work/WorkRequest.Builder#setbackoffcriteria)
    * BackOffDelay - 재시도 delay (최소 10초 이상이어야함)
    * BackOff policy - Linear / Expotential interval 시간이 어떻게 증가하는지
* Tag
  * 작업을 취소하거나 진행 상황을 관찰하기 위해서 작업마다 고유한 identifier를 가지는데, tag로 worker 그룹을 묶을 수 있음
  * 취소하기 - [WorkManager.cancelAllWorkByTag(String)](https://developer.android.com/reference/androidx/work/WorkManager?\_gl=1\*1w6xivr\*\_up\*MQ..\*\_ga\*MzY2MzExODI2LjE3MjExMTg4NTU.\*\_ga\_6HH9YJMN9M\*MTcyMTExODg1NC4xLjAuMTcyMTExODg1NC4wLjAuMA..#cancelAllWorkByTag\(java.lang.String\))
  * WorkInfo 정보 가져오기 - [WorkManager.getWorkInfosByTag(String)](https://developer.android.com/reference/androidx/work/WorkManager#getWorkInfosByTagLiveData\(java.lang.String\))
  * 태그 정보 가져오기 - [WorkInfo.getTags()](https://developer.android.com/reference/androidx/work/WorkInfo#getTags\(\)) / [ListenableWorker.getTags()](https://developer.android.com/reference/androidx/work/ListenableWorker#getTags\(\))

```kotlin
val myWorkRequest = OneTimeWorkRequestBuilder<MyWork>()
   .addTag("cleanup")
   .build()

val myWorkRequest = OneTimeWorkRequestBuilder<MyWork>()
   .setBackoffCriteria(
       BackoffPolicy.LINEAR,
       OneTimeWorkRequest.MIN_BACKOFF_MILLIS,
       TimeUnit.MILLISECONDS)
   .build()
```

## WorkRequest

* 어떻게, 언제 work를 수행할지 결정 **HOW, WHEN**
* OneTimeWorkRequest - 반복하지 않는 work&#x20;
  * Workmanager.enque / WorkManager.beginWith
* PeriodicWorkRequest - 반복하는 work
  * PeridodicWorkRequest.Builder - flexInterval 로 실행 시간 별 간격 조정 가능
* [Constraints](https://developer.android.com/develop/background-work/background-tasks/persistent/getting-started/define-work#work-constraints)
  * NetworkType&#x20;
    * CONNECTED / METERED / NOT\_REQUIRED / NOT\_ROAMING / TEMPORARILY\_UNMETERED / UNMETERED
  * BatteryNotLow
  * RequiresCahrging
  * DeviceIdle
  * StorageNotLow
  * 제약 조건이 맞지 않으면 worker 실행 안됨 & worker 실행 중에는 정지하고 조건에 맞게 되었을 때 다시 재개

```kotlin
val constraints = Constraints.Builder()
        .setRequiresBatteryNotLow(true)
        .setRequiredNetworkType(NetworkType.CONNECTED)
        .setRequiresCharging(true)
        .setRequiresStorageNotLow(true)
        .setRequiresDeviceIdle(true)
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

[`WorkManager.beginWith(OneTimeWorkRequest)`](https://developer.android.com/reference/androidx/work/WorkManager#beginWith\(androidx.work.OneTimeWorkRequest\)) [`WorkManager.beginWith(List<OneTimeWorkRequest>)`](https://developer.android.com/reference/androidx/work/WorkManager#beginWith\(java.util.List%3Candroidx.work.OneTimeWorkRequest%3E\))

작업 동시 수행 & 이어서 다음 작업 수행 등등

```kotlin
WorkManager.getInstance(myContext)
   // Candidates to run in parallel
   .beginWith(listOf(plantName1, plantName2, plantName3))
   // Dependent work (only runs after all previous work in chain)
   .then(cache)
   .then(upload)
   // Call enqueue to kick things off
   .enqueue()
```

동시 수행된 작업의 결과 합치기 - InputMerger (순서는 보장되지 않음)

* OverwritinfInputMerger
* ArrayCreatingInputMerger

```kotlin
val compressWorkRequest = OneTimeWorkRequestBuilder<CompressWorker>()
        .setInputMerger(ArrayCreatingInputMerger::class.java)
        .setConstraints(constraints)
        .build()
```

<figure><img src="../../.gitbook/assets/스크린샷 2024-07-15 오후 2.57.25.png" alt=""><figcaption></figcaption></figure>

## WorkRequest 상태 관찰하기

앱이 foreground 에 있을 때 worker가 돌고있다면, WorkInfo Livedata로 상태를 관찰하고 Ui 노출할 수 있음

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

```kotlin
// by id
workManager.getWorkInfoById(syncWorker.id) // ListenableFuture<WorkInfo>

// by name
workManager.getWorkInfosForUniqueWork("sync") // ListenableFuture<List<WorkInfo>>

// by tag
workManager.getWorkInfosByTag("syncTag") // ListenableFuture<List<WorkInfo>>
```



## 프로그래스바

[`setProgressAsync()`](https://developer.android.com/reference/androidx/work/ListenableWorker#setProgressAsync\(androidx.work.Data\))

[`CoroutineWorker`](https://developer.android.com/reference/kotlin/androidx/work/CoroutineWorker) object's [`setProgress()`](https://developer.android.com/reference/kotlin/androidx/work/CoroutineWorker#setprogress)&#x20;

```kotlin
import android.content.Context
import androidx.work.CoroutineWorker
import androidx.work.Data
import androidx.work.WorkerParameters
import kotlinx.coroutines.delay

class ProgressWorker(context: Context, parameters: WorkerParameters) :
    CoroutineWorker(context, parameters) {

    companion object {
        const val Progress = "Progress"
        private const val delayDuration = 1L
    }

    override suspend fun doWork(): Result {
        val firstUpdate = workDataOf(Progress to 0)
        val lastUpdate = workDataOf(Progress to 100)
        setProgress(firstUpdate)
        delay(delayDuration)
        setProgress(lastUpdate)
        return Result.success()
    }
}
```

```kotlin
WorkManager.getInstance(applicationContext)
    // requestId is the WorkRequest id
    .getWorkInfoByIdLiveData(requestId)
    .observe(observer, Observer { workInfo: WorkInfo? ->
            if (workInfo != null) {
                val progress = workInfo.progress
                val value = progress.getInt(Progress, 0)
                // Do something with progress information
            }
    })
```

### WorkQueires

WorkManager 2.4.0 버전 이상

enqueued jobs 의 정보를 가져오고 관찰 가능! - [`getWorkInfosLiveData()`](https://developer.android.com/reference/androidx/work/WorkManager?\_gl=1\*is6xno\*\_up\*MQ..\*\_ga\*MzY2MzExODI2LjE3MjExMTg4NTU.\*\_ga\_6HH9YJMN9M\*MTcyMTExODg1NC4xLjAuMTcyMTExOTUzMy4wLjAuMA..#getWorkInfosLiveData\(androidx.work.WorkQuery\))

```kotlin
// FAILED, CANCELLED 상태에 있는 syncTag 태그 && preProcess, sync 라는 이름의 worker 가져오기
// (name1 OR name2 OR ...) AND (tag1 OR tag2 OR ...) AND (state1 OR state2 OR ...)

val workQuery = WorkQuery.Builder
       .fromTags(listOf("syncTag"))
       .addStates(listOf(WorkInfo.State.FAILED, WorkInfo.State.CANCELLED))
       .addUniqueWorkNames(listOf("preProcess", "sync")
    )
   .build()

val workInfos: ListenableFuture<List<WorkInfo>> = workManager.getWorkInfos(workQuery)
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

## 취소 & 멈추기

```kotlin
// by id
workManager.cancelWorkById(syncWorker.id)

// by name
workManager.cancelUniqueWork("sync")

// by tag
workManager.cancelAllWorkByTag("syncTag")
```



### 참고

[https://developer.android.com/develop/background-work/background-tasks/persistent/getting-started/define-work](https://developer.android.com/develop/background-work/background-tasks/persistent/getting-started/define-work?\_gl=1\*kts4hb\*\_up\*MQ..\*\_ga\*MzY2MzExODI2LjE3MjExMTg4NTU.\*\_ga\_6HH9YJMN9M\*MTcyMTExODg1NC4xLjAuMTcyMTEyMDEwMy4wLjAuMA..)

[https://medium.com/androiddevelopers/workmanager-basics-beba51e94048](https://medium.com/androiddevelopers/workmanager-basics-beba51e94048)

[https://developer.android.com/develop/background-work/background-tasks/persistent/getting-started/define-work](https://developer.android.com/develop/background-work/background-tasks/persistent/getting-started/define-work)

[https://developer.android.com/codelabs/android-workmanager?hl=ko#11](https://developer.android.com/codelabs/android-workmanager?hl=ko#11)
