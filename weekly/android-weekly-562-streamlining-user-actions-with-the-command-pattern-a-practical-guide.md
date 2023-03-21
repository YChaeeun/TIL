# Android Weekly #562 - Streamlining User Actions with the Command Pattern, A Practical Guide

[Article](https://proandroiddev.com/streamlining-user-actions-with-the-command-pattern-a-practical-guide-72e2064b4ce7)

[Github](https://github.com/saldisobi/commandDemo)

기존 뷰 시스템에서는 viewModel 에 live data 나 state 를 정의해두고, acitivity나 fragment 에서 해당 값을 observe 관찰 해서 새로운 UI를 그리거나 작업을 수행

반면 컴포즈에서는 screen에 viewModel에 함수들을 람다로 넘겨서 동작을 수행 일반적으로 아래와 같이 코드를 작성한다

```kotlin
MyModelScreen(  
	items = item.data,  
	modifier = modifier,
	onSave = viewModel::addMyModel
)
```

하지만 코드양이 많아지면 많아질 수록 복잡도가 증가한다!

동작들 마다 람다를 넘기다 보면 20여개의 람다를 넘겨야 하는 경우도 발생할 수 있다;

```kotlin
MyModelScreen(  
	items = item.data,  
	modifier = modifier,
	onSave = viewModel::onSaveClicked,  
	onAdd = viewModel::onAddClicked,  
	onUpdate = viewModel::onTextUpdate,  
	onDelete = viewModel::onDeleteClicked,  
	onList = viewModel::onListClicked
)
```

state 와 event 를 분리한다면 가독성을 높이고 유지보수하는 데 좋은 코드가 될 것!

## Command Pattern

위 문제를 해결하기 위해서 command pattern을 적용해보자!

커맨드 패턴은 요청을 캡슐화해서 유연성을 증가시켜준다

* 요청 내역을 객체로 캡슐화 -> 서로 다른 요청 내역에 따라 매개변수화 할 수 있음
* 요청을 큐에 저장 or 로그로 기록 -> 실행 취소 작업을 지원할 수 있음

#### Receiver

* 요청 사항을 수행함
* ex) ViewModel

#### Invoker

* execute() 메소드 호출 > 커맨드 객체에게 특정 작업을 수행할 것을 요청함
* ex) Button

#### Command Object

* 요청 사항
* 특정 행동과 리시버를 연결함
  * invoker에서 execute() 호출 > Command Object에서 Receiver에 있는 메소드를 호출 > 작업 수행

#### Client

* Command Object를 생성 & Reciever 설정
* ex) Activity

## Step 1 : Set up Receiver & Command

```kotlin
interface CommandReceiver {
	fun onAddClicked()
	fun onTextUpdate(newText: String)
	fun onDeleteClicked()
	fun onListClicked()
	fun onSaveClicked(text: String)
	fun processCommand(command: Command) {
		command.execute(this)
	}
}
```

```kotlin
interface Command {
	fun execute(receiver: CommandReceiver)
}
```

```kotlin
class AddCommand : Command {
	override fun execute(receiver: CommandReceiver) {
		receiver.onAddClicked()
	}
}

class TextUpdateCommand(private val newText: String) : Command {
	override fun execute(receiver: CommandReceiver) {
		receiver.onTextUpdate(newText)
	}
}

class DeleteProductCommand : Command {
	override fun execute(receiver: CommandReceiver) {
		receiver.onDeleteClicked()
	}
}

class ListCommand() : Command {
	override fun execute(receiver: CommandReceiver) {
		receiver.onListClicked()
	}
}

class SaveCommand(private val text: String) : Command {
	override fun execute(receiver: CommandReceiver) {
		receiver.onSaveClicked(text)
	}
}
```

## Step 2 : Update ViewModel execute commands

```kotlin
class MyModelViewModel @Inject constructor(
	private val myModelRepository: MyModelRepository
) : ViewModel(), CommandReceiver {

	override fun onAddClicked() {
		viewModelScope.launch {
			Log.v(TAG, "add command")
		}
	}

	override fun onTextUpdate(newText: String) {
		viewModelScope.launch {
			Log.v(TAG, "edit command")
		}
	}
	
	override fun onDeleteClicked() {
		viewModelScope.launch {
			Log.v(TAG, "delete command")
		}
	}

	override fun onListClicked() {
		viewModelScope.launch {
			Log.v(TAG, "list command")
		}
	}

	override fun onSaveClicked(text: String) {
		viewModelScope.launch {
			myModelRepository.add(text)
		}
	}

	companion object {
		const val TAG = "MyTag"
	}
}
```

## Step 3 : Remove lambdas & pass command processor

```kotlin
// BEFORE
MyModelScreen(  
	items = item.data,  
	modifier = modifier,
	onSave = viewModel::onSaveClicked,  
	onAdd = viewModel::onAddClicked,  
	onUpdate = viewModel::onTextUpdate,  
	onDelete = viewModel::onDeleteClicked,  
	onList = viewModel::onListClicked
)

// AFTER
MyModelCommandScreen(  
	items = items.data,  
	modifier = modifier,  
	commandProcessor = viewModel::processCommand  
)
```

## Step 4 : Commands from Screen to ViewModel

```kotlin
Icon(
	modifier = Modifier.clickable { commandProcessor(AddCommand()) },
	imageVector = Icons.Filled.Add,
	contentDescription = "add"
)

Icon(
	modifier = Modifier.clickable { commandProcessor(TextUpdateCommand(nameMyModel)) },
	imageVector = Icons.Filled.Edit,
	contentDescription = "edit"
)

Icon(
	modifier = Modifier.clickable { commandProcessor(DeleteProductCommand()) },
	imageVector = Icons.Filled.Delete,
	contentDescription = "delete"
)

Icon(
	modifier = Modifier.clickable { commandProcessor(ListCommand()) },
	imageVector = Icons.Filled.List,
	contentDescription = "list"
)
```

## 주의!

모든 컴포넌트에서 사용하지말고 screen-level composable (root 뷰) 에서만 사용하기

* 그외 레벨의 컴포넌트에서는 오히려 재사용성을 해치게 될 수 있다
  * 직접 screen 상태를 건드리지 않는 컴포넌트여야 재사용하기 쉬움
