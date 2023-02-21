# 템플릿 메소드패턴 Template Method Pattern

## 템플릿 메소드 패턴

* 알고리즘의 골격을 정의
* 알고리즘의 일부 단계를 서브클래스에서 구현할 수 있음
* 알고리즘의 구조는 그대로 유지하면서 알고리즘의 특정 단계를 서브클래스에서 재정의 할 수도 있음



### 다이어그램

<figure><img src="https://user-images.githubusercontent.com/22253556/71810981-f9b82f00-30b6-11ea-9c8a-47512a75e21d.png" alt=""><figcaption></figcaption></figure>

### 사용하는 경우

* 어떤 한 알고리즘을 이루는 부분 중 변하지 않는 부분을 한 번 정의해놓고, 다양해질 수 있는 부분은 서브클래스에서 정의 할 수 있도록 남겨두고자 할 때
* 서브클래스 사이의 공통적인 행동을 추출해서 하나의 공통 클래스에 몰아넣고 중복을 피하고 싶을 때

### 장점

* 알고리즘이 한 군데에 모여있으므로 변경 사항 수정이 용이
* 코드 재사용

### 후크 Hook

* 추상 클래스에서 선언되지만 기본적인 내용만 구현되어 있거나 아무 코드도 들어있지 않은 메소드
* 서브클래스에서 후크를 사용하려면 오버라이드 하면 됨

### 디자인 원칙

할리우드 원칙 Hollywood Principle

* 먼저 연락하지 마! 내가 연락할게
* 부모 클래스는 서브클래스에 정의된 연산을 호출할 수는 있지만, 반대 방향의 호출은 안된다

#### 장점

* 의존성 부패를 방지할 수 있다
  * 의존성 부패 : 의존성이 복잡하게 꼬여있는 상황을 의미



\-----

```kotlin
abstract class CaffeineBeverage {

	fun prepareBeverage() {
		boilWater()
		brew()
		if (doesCustomerWantsCondiment()) {
			addCondiments()
		}
		pourInCup()
	}

	private fun boilWater() {
		println("물 끓이기")
	}

	protected abstract fun brew()
	protected abstract fun addCondiments()

	private fun pourInCup() {
		println("컵에 따르기")
	}

	protected open fun doesCustomerWantsCondiment(): Boolean {
		return true
	}
}
```

```kotlin
class Tea : CaffeineBeverage() {

	override fun brew() {
		println("찻잎 우리기")
	}

	override fun addCondiments() {
		println("레몬 추가하기")
	}
}
```

```kotlin
class CoffeeWithHook : CaffeineBeverage() {

	override fun brew() {
		println("필터로 커피 우리기")
	}

	override fun addCondiments() {
		println("설탕과 우유 추가하기")
	}

	override fun doesCustomerWantsCondiment(): Boolean {
		val answer = getUserInput()
		return answer.lowercase(Locale.ROOT).startsWith("y")
	}

	private fun getUserInput(): String {
		print("커피에 우유와 설탕을 넣을까요? (y/n) : ")

		return readLine() ?: return "no"
	}
}
```

```kotlin
fun main() {
	val tea = Tea()
	tea.prepareBeverage()

	val coffeeWithHook = CoffeeWithHook()
	coffeeWithHook.prepareBeverage()
}

/**
물 끓이기
찻잎 우리기
레몬 추가하기
컵에 따르기
물 끓이기

필터로 커피 우리기
커피에 우유와 설탕을 넣을까요? (y/n) : y
설탕과 우유 추가하기
컵에 따르기
**/
```
