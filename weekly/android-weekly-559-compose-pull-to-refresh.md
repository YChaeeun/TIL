# Android Weekly #559 - Compose로 Pull-to-Refresh 구현하기

[https://medium.com/google-developer-experts/effortlessly-add-pull-to-refresh-to-your-android-app-with-jetpack-compose-4c8b218a9beb](https://medium.com/google-developer-experts/effortlessly-add-pull-to-refresh-to-your-android-app-with-jetpack-compose-4c8b218a9beb)

## dependency 추가

```kotlin
dependencies {
	implementation "androidx.compose.material:material:1.4.0-beta01"
}
```

* 현재 안정화 버전에 버그가 있음!
  * 1.3.1 버전 - 리프레시 후 인디케이터가 위로 가지 않는 버그가 있음

## 리프레시 관찰

* Flow나 Livedata 로 상태를 추적/관찰
  * 리프레시가 언제 시작하고 끝났는지
  *

```kotlin
private val _isRefreshing = MutableStateFlow(false)
val isRefreshing: StateFlow<Boolean>
	get() = _isRefreshing.asStateFlow()

private val _item = MutableStateFlow<Fox?>(null)
val item: StateFlow<Fox?>
	get() = _item.asStateFlow()

init {
	refresh()
}

fun refresh() {
	// pull-to-refresh api 가 _isRefreshing 값을 true로 변경해줄 것이기 때문에
	// 여기서 수동으로 값 set 하지 않기

	viewModelScope.launch {
		val foxList = mutableListOf<Fox>()
		async(Dispatcher.IO) { 
		// 여러 동작이 있을 경우, 모든 동작이 완료되고 _isRefreshing 값을 false로 만들기 위해 async/await 사용
			_item.emit(foxService.getRandomFox().await())
		}
		_isRefreshing.emit(false) // 리프레시 동작이 끝나면 값을 false로 변경
	}
}
```

### CollectAsStateWithLifecycle

life-cycle을 알면서 상태를 관찰할 수 있음

```kotlin
val viewModel: MainViewModel = hiltViewModel()

val refreshing by viewModel.isRefreshing.collectAsStateWithLifecycle()

val pullRefreshState = rememberPullRefreshState(refreshing, { viewModel.refresh() })

val foxData: Fox? by viewModel.item.collectAsStateWithLifecycle()
```

### 동작

* Modifier.pullRefresh(PullRefreshState) 를 추가
* 스크롤 동작
  * 추가하고 싶다면 LazyColumn
  * 사용하고 싶지 않다면 Column 이나 Box 사용
    * 단, 스크롤이 안되는 컴포저블을 사용할 때는 Modifier.verticalScroll 로 스크롤 상태를 알려줘야 pull action이 동작함

```kotlin
Box(
	Modifier
		.pullRefresh(pullRefreshState)
		.verticalScroll(rememberScrollState())
) {

	// Display data

	PullRefreshIndicator(isRefreshing, pullRefreshState, Modifier.align(Alignment.TopCenter))

}
```

### 전체 코드

```kotlin
val viewModel: MainViewModel = hiltViewModel()

val foxData: Fox? by viewModel.item.collectAsStateWithLifecycle()

val isRefreshing by viewModel.isRefreshing.collectAsStateWithLifecycle()

val pullRefreshState = rememberPullRefreshState(isRefreshing, { viewModel.refresh() })

Box(
	Modifier
		.pullRefresh(pullRefreshState)
		.verticalScroll(rememberScrollState()) // LazyColumn 쓰면 없어도 됨
) {

	foxData?.image?.let {
		AsyncImage(
			model = it,
			contentDescription = null,
			contentScale = ContentScale.FillWidth,
			modifier = Modifier.fillMaxWidth()
		)
	}

	PullRefreshIndicator(
		refreshing = isRefreshing,
		state = pullRefreshState,
		modifier = Modifier.align(Alignment.TopCenter)
	)
}
```
